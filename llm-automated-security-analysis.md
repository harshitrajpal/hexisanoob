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





