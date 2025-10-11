# Page 1

```python
#!/usr/bin/env python3
"""
update_and_post_signed_with_poll.py

- Read prompt.txt and upload to s3://aaa-manually-created-pentester-website/example-website/index.html
- Load a multi-line JSON payload from payload.json (if present) OR use an inline Python dict
- Send a POST signed with SigV4 to API Gateway using requests + requests-aws4auth
- Extract 'jobId' from the JSON response
- Wait ~60 seconds, then GET POST_URL/<jobId> (signed) and capture the response
- Write CSV with jobId, payload_json, and website_output
"""

import os
import json
import csv
import sys
import time
from typing import Optional, Any, Dict

import boto3
from botocore.exceptions import BotoCoreError, ClientError
import requests
from requests_aws4auth import AWS4Auth

# -------------------------
# Configuration - edit as needed
# -------------------------
S3_BUCKET = "aaa-manually-created-pentester-website"
S3_KEY = "example-website/index.html"
PROMPT_FILE = "prompt.txt"

PAYLOAD_FILE = "payload.json"  # optional; set to None to always use inline
CSV_OUTPUT = "job_results.csv"

# The API base URL to POST to and later GET from (GET will use /<jobId> appended)
POST_URL = "https://example.com"  # <-- change to the real endpoint

# Region & service for SigV4
AWS_REGION = "us-east-1"
AWS_SERVICE = "execute-api"  # for API Gateway

# Inline payload (multi-line as Python dict)
PAYLOAD_OBJ = {
    "creditApplication": {
        "applicationId": "00000000-0000-0000-0000-000000000000",
        "awsAccountId": "123456789012",
        "companyName": "Amazon",
        "companyDescription": "Lorem ipsum dolor sit amet, consectetur...",
        "startupIndustries": ["ACCOUNTING"]
    },
    "websiteUrl": "https://aws.amazon.com/startups"
}

# How long to wait (seconds) before polling the GET endpoint
POLL_DELAY_SECONDS = 60

# Request timeout seconds
REQUEST_TIMEOUT = 30

# -------------------------
# Helpers
# -------------------------
def read_prompt_file(path: str) -> str:
    if not os.path.exists(path):
        raise FileNotFoundError(f"prompt file not found: {path}")
    with open(path, "r", encoding="utf-8") as f:
        return f.read()


def upload_to_s3(bucket: str, key: str, body: str, region_name: str = AWS_REGION) -> Dict[str, Any]:
    s3 = boto3.client("s3", region_name=region_name)
    try:
        resp = s3.put_object(
            Bucket=bucket,
            Key=key,
            Body=body.encode("utf-8"),
            ContentType="text/html"
        )
        return resp
    except (BotoCoreError, ClientError) as e:
        raise RuntimeError(f"Failed to upload to S3: {e}")


def load_payload(payload_file: Optional[str]) -> Dict:
    if payload_file and os.path.exists(payload_file):
        with open(payload_file, "r", encoding="utf-8") as f:
            data = json.load(f)
        return data
    return PAYLOAD_OBJ


def get_aws_auth(region: str = AWS_REGION, service: str = AWS_SERVICE) -> AWS4Auth:
    session = boto3.Session()
    credentials = session.get_credentials()
    if credentials is None:
        raise RuntimeError("Unable to locate AWS credentials (environment, shared credentials file, or IAM role).")
    frozen = credentials.get_frozen_credentials()
    if not frozen.access_key or not frozen.secret_key:
        raise RuntimeError("Incomplete AWS credentials.")
    awsauth = AWS4Auth(frozen.access_key, frozen.secret_key, region, service, session_token=frozen.token)
    return awsauth


def post_signed(url: str, payload_obj: Dict, awsauth: AWS4Auth, timeout: int = REQUEST_TIMEOUT) -> requests.Response:
    headers = {"Content-Type": "application/json"}
    resp = requests.post(url, auth=awsauth, json=payload_obj, headers=headers, timeout=timeout)
    resp.raise_for_status()
    return resp


def parse_job_id_from_response_json(parsed_json: Any) -> Optional[str]:
    """
    Given a parsed JSON object (dict or otherwise) try to find 'jobId' top-level or shallow-nested.
    """
    if isinstance(parsed_json, dict):
        if "jobId" in parsed_json:
            return parsed_json["jobId"]
        for v in parsed_json.values():
            if isinstance(v, dict) and "jobId" in v:
                return v["jobId"]
    return None


def parse_job_id(resp: requests.Response) -> Optional[str]:
    try:
        parsed = resp.json()
    except ValueError:
        return None
    return parse_job_id_from_response_json(parsed)


def poll_job_result(job_id: str, base_url: str, awsauth: AWS4Auth, delay_seconds: int = POLL_DELAY_SECONDS, timeout: int = REQUEST_TIMEOUT) -> str:
    """
    Wait roughly delay_seconds, then GET base_url/<job_id> (signed) and return the response body.
    Returns a string (JSON pretty or raw text or error message).
    """
    if not job_id:
        return "No jobId provided; skipping GET."

    # Wait
    print(f"Waiting {delay_seconds} seconds before polling result for jobId={job_id} ...")
    time.sleep(delay_seconds)

    get_url = base_url.rstrip("/") + "/" + job_id
    print(f"GET {get_url}")

    try:
        resp = requests.get(get_url, auth=awsauth, timeout=timeout)
        resp.raise_for_status()
    except requests.HTTPError as e:
        body = getattr(e, "response", None).text if getattr(e, "response", None) is not None else ""
        return f"HTTP error during GET: {e} ; body: {body}"
    except Exception as e:
        return f"Error during GET request: {e}"

    # Try to return pretty JSON if possible, else raw text
    try:
        parsed = resp.json()
        return json.dumps(parsed, indent=2)
    except ValueError:
        return resp.text


def write_csv(output_file: str, job_id: Optional[str], payload_obj: Dict, website_output: str):
    header = ["jobId", "payload_json", "website_output"]
    payload_str = json.dumps(payload_obj, indent=2)
    with open(output_file, "w", newline="", encoding="utf-8") as csvf:
        writer = csv.writer(csvf)
        writer.writerow(header)
        writer.writerow([job_id if job_id is not None else "", payload_str, website_output])


# -------------------------
# Main
# -------------------------
def main():
    # 1) Read prompt.txt
    try:
        prompt_content = read_prompt_file(PROMPT_FILE)
    except Exception as e:
        print(f"Error reading prompt file '{PROMPT_FILE}': {e}", file=sys.stderr)
        sys.exit(1)

    # 2) Upload prompt content to S3 as index.html inside example-website folder
    try:
        print(f"Uploading {PROMPT_FILE} -> s3://{S3_BUCKET}/{S3_KEY} ...")
        resp = upload_to_s3(S3_BUCKET, S3_KEY, prompt_content)
        print("S3 put_object response metadata:", resp.get("ResponseMetadata", {}))
    except Exception as e:
        print("S3 upload failed:", e, file=sys.stderr)
        sys.exit(1)

    # 3) Load payload (file or inline)
    try:
        payload_obj = load_payload(PAYLOAD_FILE)
    except Exception as e:
        print("Failed to load payload:", e, file=sys.stderr)
        sys.exit(1)

    print("Payload (pretty):")
    print(json.dumps(payload_obj, indent=2))

    # 4) Get AWS sigv4 auth object
    try:
        awsauth = get_aws_auth()
    except Exception as e:
        print("Failed to obtain AWS auth:", e, file=sys.stderr)
        sys.exit(1)

    # 5) POST signed request
    try:
        print(f"POSTing signed request to {POST_URL} ...")
        response = post_signed(POST_URL, payload_obj, awsauth)
        print("Response status:", response.status_code)
        try:
            print("Response JSON (pretty):")
            print(json.dumps(response.json(), indent=2))
        except Exception:
            print("Response text:", response.text)
    except requests.HTTPError as e:
        print("HTTP error during POST:", e, file=sys.stderr)
        if hasattr(e, 'response') and e.response is not None:
            print("Response body:", e.response.text, file=sys.stderr)
        sys.exit(1)
    except Exception as e:
        print("Error during POST:", e, file=sys.stderr)
        sys.exit(1)

    # 6) Extract jobId
    job_id = parse_job_id(response)
    if job_id is None:
        print("Warning: could not parse 'jobId' from response JSON. Will skip polling GET.", file=sys.stderr)
        website_output = "Skipped GET because jobId could not be parsed from POST response."
    else:
        print("Parsed jobId:", job_id)
        # 7) Wait ~60s and GET base_url/<jobId>
        website_output = poll_job_result(job_id, POST_URL, awsauth, delay_seconds=POLL_DELAY_SECONDS)
        print("Website output (or error):")
        print(website_output)

    # 8) Write CSV with jobId, payload and website_output
    try:
        write_csv(CSV_OUTPUT, job_id, payload_obj, website_output)
        print(f"Wrote CSV to {CSV_OUTPUT}")
    except Exception as e:
        print("Failed to write CSV:", e, file=sys.stderr)
        sys.exit(1)


if __name__ == "__main__":
    main()

```
