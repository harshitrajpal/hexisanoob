---
description: Uses https://github.com/NVIDIA/garak
---

# LLM Automated Security Analysis

_Generative AI Red-teaming & Assessment Kit - GARAK_

`garak` checks if an LLM can be made to fail in a way we don't want. `garak` probes for hallucination, data leakage, prompt injection, misinformation, toxicity generation, jailbreaks, and many other weaknesses. If you know `nmap` or `msf` / Metasploit Framework, garak does somewhat similar things to them, but for LLMs.

`garak` focuses on ways of making an LLM or dialog system fail. It combines static, dynamic, and adaptive probes to explore this.



Installation

Due to the intricacies of packages and to make sure our system packages don't break we'll use conda

{% embed url="https://docs.conda.io/projects/conda/en/stable/user-guide/install/index.html" %}

Use this to identify your env [https://repo.anaconda.com/archive/](https://repo.anaconda.com/archive/)

I'll use "[Anaconda3-2025.06-1-Linux-x86\_64.sh](https://repo.anaconda.com/archive/Anaconda3-2025.06-1-Linux-x86_64.sh)" on my Linux machine



Just install it with all default options

***

Once conda is installed, proceed with garak installation



`conda create --name garak "python>=3.10,<=3.12"`\
`conda activate garak`\
`git clone` [`https://github.com/NVIDIA/garak.git`](https://github.com/NVIDIA/garak.git)

`cd garak`\
`python -m pip install -e .`

Once installed, confirm installation with

`garak -h`

***

Now garak can connect to different LLM interfaces. Most common is the HTTP REST API endpoint that returns JSON/plaintext output.

let's assume an LLM is replying on `/api/v1/ai/chat` endpoint on a host "example.com"

Let's assume the API request looks like the following:

```http
POST /api/v1/ai/chat HTTP/1.1
Host: example.com
Cookie: Bearer AUTH TOKEN
Content-Length: 349
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/138.0.0.0 Safari/537.36
Content-Type: application/json
Accept: */*
Origin: example.com
Accept-Encoding: gzip, deflate, br
Priority: u=1, i
Connection: keep-alive

{"datetime":"2025-09-25T19:58:25.344+00:00","domain_id":"example.com","user_id":"1","content_id":"randomcontent","item_id":"123456789","prompt":"query","question":"What is the weather in jersey"}
```

Let's assume the response body looks like:

```
HTTP/1.1 200 OK
Date: Thu, 25 Sep 2025 19:58:26 GMT
Content-Type: application/json
Content-Length: 518
Connection: keep-alive
Server: nginx
Access-Control-Allow-Methods: POST, GET, OPTIONS

{"version": "1", "response": [{"text": "### Weather in Jersey\n\nUnfortunately, the provided content does not contain information about the weather in Jersey. If you are looking for weather updates, it is recommended to check a reliable weather website or app for the most current information.", "rts": 0.8001093830025638, "logged": []}], "model": "gpt-35-turbo-16k-1106"}
```

You would manually need to create a JSON config file. You can refer to the docs here: [https://reference.garak.ai/en/latest/garak.generators.rest.html](https://reference.garak.ai/en/latest/garak.generators.rest.html)

For our case, config becomes like:

**api\_web\_config.json**

```json
{
   "rest": {
      "RestGenerator": {
         "name": "Example Content Copilot",
         "uri": "https://example.com/api/v1/ai/chat",
         "method": "post",
         "headers": {
            "Cookie": "Bearer AUTH TOKEN",
            "Content-Type": "application/json",
	    "Accept": "*/*",
	    "User-Agent": "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/138.0.0.0 Safari/537.36"
	    "Origin": "example.com",
	    "Referer": "https://hsptpentest.latest.highspot.com/items/68d0267d9d2908070669c7de?lfrm=shp.1",
	    "Accept-Encoding": "gzip, deflate, br",
	    "Priority": "u=1, i",
            "Connection": "keep-alive"
	 },
         "req_template_json_object": {
            "datetime":"2025-09-25T15:49:27.950+00:00",
            "domain_id":"example.com",
            "user_id":"1",
            "content_id":"randomcontent",
            "item_id":"123456789",
            "prompt":"query",
            "question":"$INPUT",
         },
         "response_json": true,
         "response_json_field": "$.response[0].text",
         "skip_codes": [500,504,422],
         "request_timeout": 3000
      }
   }
}

```

Now the two main fields we need to focus in the config file above is identifying which parameter user sends their query in and which parameter does the LLM respond back in.

Here, "question" parameter in the request holds user's query and "response.text" contains LLM response

So, these two fields are specially marked in **api\_web\_config.json**

**"$INPUT"** tells garak where to inject prompts in for testing. Put this in the user controlled param for LLM query in you case.

"**response\_json\_field**" tells garak where to look for LLM response to analyze whether attack vectors worked or not. You can define the specific parameter using basic JSON object definition syntax. For example, here response is in the first field "text" encapsulated by "response" object so we defined "$.response\[0].text"



Once done you are free to run garak!

`garak --model_type rest -G api_web_config.json`

You can ploy with speed throttles as well

`garak --model_type rest -G api_web_config.json --parallel_attempts 20`

***

Test Garak



`garak --model_type test.Blank --probes test.Test`

`garak --model_type rest -G api_web_config.json --probes test.Test`

Let's say you only want specific tests like prompt injections. You can use "garak --probes" to list all the different probes

`garak --model_type rest -G api_web_config.json --probes promptinject --parallel_attempts 20`

***

**Reading the report**

```python
#!/usr/bin/env python3
import json, csv, gzip, io, os, sys, argparse, re
from typing import Any, Dict, Iterable, Tuple, Union, List

def open_maybe_gzip(path: str, mode: str, encoding: str = "utf-8"):
    """
    Open plain text or .gz files transparently.
    mode: "rt" for read text, "wt" for write text
    """
    if path.endswith(".gz"):
        # newline='' is important for csv writer on Windows; use text mode with encoding.
        return io.TextIOWrapper(gzip.open(path, mode.replace("t", "") ), encoding=encoding, newline="")
    else:
        return open(path, mode, encoding=encoding, newline="")

def flatten_json(obj: Any, parent_key: str = "", sep: str = ".",
                 list_join: str = "; ", max_list_elems: int = 0) -> Dict[str, str]:
    """
    Flatten nested dicts into dotted keys. Lists are joined with list_join.
    - max_list_elems=0 means join all elements; if >0, truncate and append " â€¦" marker.
    Scalar values are converted to strings (except None -> "").
    """
    out: Dict[str, str] = {}

    def _stringify(v: Any) -> str:
        if v is None:
            return ""
        if isinstance(v, (str, int, float, bool)):
            return str(v)
        # For non-scalar leftover (e.g., dict in a list), dump compact JSON
        return json.dumps(v, ensure_ascii=False, separators=(",", ":"))

    def _flatten(x: Any, prefix: str = ""):
        if isinstance(x, dict):
            for k, v in x.items():
                key = f"{prefix}{sep}{k}" if prefix else str(k)
                _flatten(v, key)
        elif isinstance(x, list):
            if len(x) == 0:
                out[prefix] = ""
                return
            # Join elements after converting each to a safe string
            items = []
            limit = len(x) if max_list_elems <= 0 else min(max_list_elems, len(x))
            for i in range(limit):
                items.append(_stringify(x[i]))
            if max_list_elems > 0 and len(x) > max_list_elems:
                items.append("â€¦")
            out[prefix] = list_join.join(items)
        else:
            out[prefix] = _stringify(x)

    _flatten(obj, parent_key)
    return out

def iter_jsonl(path: str, encoding: str = "utf-8") -> Iterable[Tuple[int, Any]]:
    """Yield (line_number, parsed_json) for each non-empty line in a JSONL/NDJSON file (or .gz)."""
    with open_maybe_gzip(path, "rt", encoding=encoding) as f:
        for i, line in enumerate(f, start=1):
            line = line.strip()
            if not line:
                continue
            try:
                yield i, json.loads(line)
            except json.JSONDecodeError as e:
                raise SystemExit(f"JSON parse error at line {i}: {e}")

def collect_columns(path: str, sep: str, list_join: str, max_list_elems: int, encoding: str) -> List[str]:
    """First pass: discover the union of flattened keys across all rows."""
    cols: set = set()
    for _, obj in iter_jsonl(path, encoding=encoding):
        flat = flatten_json(obj, sep=sep, list_join=list_join, max_list_elems=max_list_elems)
        cols.update(flat.keys())
    # Stable order: sorted case-insensitively
    return sorted(cols, key=lambda s: s.lower())

def write_csv(jsonl_path: str, csv_path: str, sep: str = ".", list_join: str = "; ",
              max_list_elems: int = 0, encoding: str = "utf-8", fields: list | None = None,
              wildcard_join: str = "; ") -> int:
    """Convert JSONL -> CSV. Returns number of rows written."""
        # If specific fields are requested, we use them directly (no discovery pass).
    if fields:
        columns = fields
    else:
        columns = collect_columns(jsonl_path, sep=sep, list_join=list_join, max_list_elems=max_list_elems, encoding=encoding)
    if not columns:
        # No rows -> create an empty CSV with no header
        with open_maybe_gzip(csv_path, "wt", encoding=encoding) as out_f:
            pass
        return 0

    with open_maybe_gzip(csv_path, "wt", encoding=encoding) as out_f:
        writer = csv.DictWriter(out_f, fieldnames=columns)
        writer.writeheader()
        count = 0
        for _, obj in iter_jsonl(jsonl_path, encoding=encoding):
            if fields:
                row = {col: extract_path(obj, col, wildcard_join) for col in columns}
            else:
                flat = flatten_json(obj, sep=sep, list_join=list_join, max_list_elems=max_list_elems)
                row = {col: flat.get(col, "") for col in columns}
            writer.writerow(row)
            count += 1
    return count


def _iterate_wild(obj, parts):
    """Yield values by following parts; supports '*' to expand lists/dicts."""
    if not parts:
        yield obj
        return
    head, *tail = parts
    if head == "*":
        # expand lists or dicts
        if isinstance(obj, list):
            for item in obj:
                yield from _iterate_wild(item, tail)
        elif isinstance(obj, dict):
            for item in obj.values():
                yield from _iterate_wild(item, tail)
        else:
            # not expandable; stop
            return
    else:
        # normal key or numeric index
        nxt = None
        if isinstance(obj, dict):
            if head in obj:
                nxt = obj[head]
            else:
                # allow sloppy key variants like spaces vs underscores
                alt = head.replace(" ", "") if " " in head else head.replace("", " ")
                if alt in obj:
                    nxt = obj[alt]
        elif isinstance(obj, list):
            try:
                idx = int(head)
                if 0 <= idx < len(obj):
                    nxt = obj[idx]
            except Exception:
                return
        if nxt is None:
            return
        yield from _iterate_wild(nxt, tail)

def extract_path(obj, path: str, joiner: str = "; "):
    """
    Extract value(s) from obj by a dot path with '*' wildcard.
    Returns a string (joined if multiple). If nothing is found, returns ''.
    Example paths: 'uuid', 'probe_classname', 'prompt.turns..content.text', 'outputs..text'
    """
    parts = path.split(".") if path else []
    vals = list(_iterate_wild(obj, parts))
    if not vals:
        return ""
    # stringify each value
    formatted = []
    for v in vals:
        if v is None:
            formatted.append("")
        elif isinstance(v, (str, int, float, bool)):
            formatted.append(str(v))
        else:
            formatted.append(json.dumps(v, ensure_ascii=False, separators=(",", ":")))
    # Deduplicate while preserving order
    seen = set()
    uniq = []
    for s in formatted:
        if s not in seen:
            seen.add(s)
            uniq.append(s)
    return joiner.join(uniq)


def main(argv: List[str] = None) -> int:
    p = argparse.ArgumentParser(
        description="Convert JSONL/NDJSON to CSV with nested fields flattened, or extract specific fields with wildcards."
    )
    p.add_argument("input", help="Path to .jsonl/.ndjson (plain or .gz)")
    p.add_argument("output", help="Path to output .csv (plain or .gz)")
    p.add_argument("--sep", default=".", help="Key separator for nested objects (default: '.')")
    p.add_argument("--list-join", default="; ", help="Join string used for list values")
    p.add_argument("--max-list-elems", type=int, default=0,
                   help="If >0, truncate lists to this many elements and append 'â€¦'")
    p.add_argument("--encoding", default="utf-8", help="Text encoding for input/output (default utf-8)")
    p.add_argument("--fields", nargs="", help="Dot paths to extract (supports '' to collect from lists)")
    p.add_argument("--join-list", dest="wildcard_join", default="; ", help="Join string used when a path collects multiple values via '*' (default '; ')")

    args = p.parse_args(argv)

    rows = write_csv(
        jsonl_path=args.input,
        csv_path=args.output,
        sep=args.sep,
        list_join=args.list_join,
        max_list_elems=args.max_list_elems,
        encoding=args.encoding,
        fields=args.fields,
        wildcard_join=args.wildcard_join,
    )
    print(f"Wrote {rows} rows to {args.output}")
    return 0

if _name_ == "_main_":
    raise SystemExit(main())
```



