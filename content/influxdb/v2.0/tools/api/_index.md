---
title: Working with the InfluxDB API
seotitle: Use the InfluxDB API
description: Interact with InfluxDB 2.0 using a rich API for writing and querying data and more.
weight: 103
menu:
  influxdb_2_0:
    name: Working with the API
    parent: Tools & integrations
---

InfluxDB offers a rich API that you can use to create applications.
The following pages offer general guides to the most commonly used API methods.

For detailed documentation on the entire API, see [InfluxDBv2 API Documentation](/influxdb/v2.0/reference/api/#influxdb-v2-api-documentation).

{{% note %}}
If you are interacting with InfluxDB 1.x, see the [1.x compatibility API](/influxdb/v2.0/reference/api/influxdb-1x/).
{{% /note %}}

Use the following API endpoints to write and query data:

## Write API

[Write data to InfluxDB](/influxdb/v2.0/write-data/developer-tools/api/) using an HTTP request to the InfluxDB API `/write` endpoint.

## Query API

[Query from InfluxDB](/influxdb/v2.0/query-data/execute-queries/influx-api/) using an HTTP request to the `/query` endpoint.


## Create a token with the API

The following example shows how to create a read-only token with a simple Python script.

(See also [Client library]().)

### Example

```python
import requests
import json
import os
import secrets

flask_token = os.getenv("INFLUXDB_TOKEN")
flask_orgid = os.getenv("INFLUXDB_ORGID")

protocol="http://"
domain="us-west-2-1.aws.cloud2.influxdata.com"
api_path="/api/v2/authorizations/"
url = protocol + domain + api_path

headers = {"Authorization": "Token " + flask_token}
namehex = secrets.token_hex(3)
payload = {
    "orgID": flask_orgid,
    "description": "flask read/write token-" + namehex,
    "permissions": [
        {
            "action": "read",
            "resource": {
                "type": "buckets",
                "orgID": flask_orgid,
            },
        },
    ],
}

r = requests.post(url, headers=headers, json=payload)
pretty_json = json.loads(r.text)
# To debug:
# print(json.dumps(pretty_json, indent=2))

authID = pretty_json["id"]
token =  pretty_json["token"]
print("Your new token:\n ", token)
print("Keep it secret! Keep it safe!")
```
