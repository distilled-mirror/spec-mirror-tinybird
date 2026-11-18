---
last_updated: "2026-11-04T12:15:44.000Z"
title: Sink Pipes API Reference
meta:
    description: The Sink Pipes API allows you to create, delete, schedule, and trigger Sink Pipes.
headingMaxLevels: 2
---

# Sink Pipes API

{% snippet title="api-region-reminder" /%}

Use the Sink Pipes API to create, delete, schedule, and trigger Sink Pipes.

## POST /v0/pipes/{pipe_id}/nodes/{node_id}/sink

Set the Pipe as a Sink Pipe, optionally scheduled.

Required token permission is `PIPES:CREATE`.

### Restrictions

- You can set only one schedule per Sink Pipe.
- You can't set a Sink Pipe if the Pipe is already materializing. You must unlink the Materialization first.
- You can't set a Sink Pipe if the Pipe is already an API Endpoint. You must unpublish the Endpoint first.
- You can't set a Sink Pipe if the Pipe is already copying. You must unset the copy first.
- Sink Pipes aren't available on the Free plan. The request returns a `400` error.
- In Tinybird Forward, this endpoint returns a `403` error. Define Sink Pipes in your datafiles and deploy them.

### Example

```shell {% title="Setting the Pipe as a Sink Pipe" %}
curl \
  -X POST "{% user("apiHost") %}/v0/pipes/:pipe/nodes/:node/sink" \
  -H "Authorization: Bearer <PIPES:CREATE token>" \
  -d "connection=my_connection_name" \
  -d "path=s3://bucket-name/prefix" \
  -d "file_template=exported_file_template" \
  -d "format=csv" \
  -d "compression=gz" \
  -d "schedule_cron=0 */1 * * *" \
  -d "write_strategy=new"
```

### Request parameters

{% table %}
  * Key
  * Type
  * Description
  ---
  * connection
  * String
  * Name of the connection to holding the credentials to run the sink
  ---
  * path
  * String
  * Object store prefix into which the sink writes data
  ---
  * file_template
  * String
  * File template string. See [file template](/classic/publish-data/sinks/s3-sink#file-template) for more details
  ---
  * format
  * String
  * Optional. Format of the exported files. Default: CSV
  ---
  * compression
  * String
  * Optional. Compression of the output files. Default: None
  ---
  * schedule_cron
  * String
  * Optional. The sink's execution schedule, in crontab format.
  ---
  * service
  * String
  * Optional. Sink service. Set to `kafka` to create a Kafka Sink; otherwise the Sink writes to object storage.
  ---
  * topic
  * String
  * Optional. Kafka topic to write to. Used by Kafka Sinks.
  ---
  * ignore_sql_errors
  * Boolean
  * Optional. Default: `false`. Set to `true` to skip the validation of the Node SQL when creating the Sink.
  ---
  * dry_run
  * Boolean
  * Optional. Default: `false`. Set to `true` to validate the request without creating the Sink. Returns `{"dry_run_passed": true}`.
  ---
  * write_strategy
  * String
  * Optional. Default: `create_new`. The sink's write strategy for filenames already existing in the bucket. Values: `create_new`, `replace`; `create_new` adds a new file with a suffix, while `replace` replaces the existent file. The legacy values `new` and `truncate` are still accepted.
{% /table %}

### Successful response example

```json
{
    "id": "t_529f46626c324674b3a84cd820ac2649",
    "name": "p_test",
    "description": null,
    "endpoint": null,
    "created_at": "2024-01-18 12:57:36.503834",
    "updated_at": "2024-01-18 13:01:21.435012",
    "parent": null,
    "type": "sink",
    "last_commit": {
        "content_sha": "",
        "path": "",
        "status": "changed"
    },
    "sink_node": "t_6e8afdb8c691459b80e16541433f951b",
    "schedule": {
        "timezone": "Etc/UTC",
        "cron": "0 */1 * * *",
        "status": "running"
    },
    "nodes": [
        {
            "id": "t_6e8afdb8c691459b80e16541433f951b",
            "name": "p_test_0",
            "sql": "SELECT * FROM test",
            "description": null,
            "materialized": null,
            "cluster": null,
            "tags": {},
            "created_at": "2024-01-18 12:57:36.503843",
            "updated_at": "2024-01-18 12:57:36.503843",
            "version": 0,
            "project": null,
            "result": null,
            "ignore_sql_errors": false,
            "node_type": "sink",
            "dependencies": [
                "test"
            ],
            "params": []
        }
    ]
}
```

### Response codes

{% table %}
  * Code
  * Description
  ---
  * 200
  * OK
  ---
  * 404
  * Pipe, Node, or data connector not found, bucket doesn't exist
  ---
  * 403
  * Limit reached, Query includes forbidden keywords, Pipe is already a Sink Pipe, can't assume role
  ---
  * 401
  * Invalid credentials (from connection)
  ---
  * 400
  * Invalid or missing parameters, bad ARN role, invalid region name
{% /table %}

## DELETE /v0/pipes/{pipe_id}/nodes/{node_id}/sink

Removes the Sink from the Pipe. This doesn't delete the Pipe nor the Node, only the sink configuration and any associated settings.

### Example

```shell
curl \
  -X DELETE "{% user("apiHost") %}/v0/pipes/$1/nodes/$2/sink" \
  -H "Authorization: Bearer <PIPES:CREATE token>"

Successful response example
{
    "id": "t_529f46626c324674b3a84cd820ac2649",
    "name": "p_test",
    "description": null,
    "endpoint": null,
    "created_at": "2024-01-18 12:57:36.503834",
    "updated_at": "2024-01-19 09:27:12.069650",
    "parent": null,
    "type": "default",
    "last_commit": {
        "content_sha": "",
        "path": "",
        "status": "changed"
    },
    "nodes": [
        {
            "id": "t_6e8afdb8c691459b80e16541433f951b",
            "name": "p_test_0",
            "sql": "SELECT * FROM test",
            "description": null,
            "materialized": null,
            "cluster": null,
            "tags": {},
            "created_at": "2024-01-18 12:57:36.503843",
            "updated_at": "2024-01-19 09:27:12.069649",
            "version": 0,
            "project": null,
            "result": null,
            "ignore_sql_errors": false,
            "node_type": "standard",
            "dependencies": [
                "test"
            ],
            "params": []
        }
    ],
    "url": "https://api.split.tinybird.co/v0/pipes/p_test.json"
}
```

### Response codes

{% table %}
  * Code
  * Description
  ---
  * 200
  * OK
  ---
  * 404
  * Pipe, Node, or data connector not found
  ---
  * 403
  * Limit reached, Query includes forbidden keywords, Pipe is already a Sink Pipe
  ---
  * 400
  * Invalid or missing parameters, Pipe isn't a Sink Pipe
{% /table %}

## POST /v0/pipes/{pipe_id}/sink
Triggers the Sink Pipe and creates a sink job. You can override some sink settings for this execution.

### Example

```shell {% title="Trigger a Sink Pipe with some overrides" %}
curl \
  -X POST "{% user("apiHost") %}/v0/pipes/p_test/sink" \
  -H "Authorization: Bearer <PIPES:READ token>" \  
  -d "file_template=export_file" \
  -d "format=csv" \
  -d "compression=gz" \
  -d "write_strategy=replace" \
  -d {key}={val}
```

### Request parameters

{% table %}
  * Key
  * Type
  * Description
  ---
  * file_template
  * String
  * File template string. See [file template](/classic/publish-data/sinks/s3-sink#file-template) for more details
  ---
  * format
  * String
  * Optional. Format of the exported files. Default: CSV
  ---
  * compression
  * String
  * Optional. Compression of the output files. Default: None
  ---
  * write_strategy
  * String
  * Optional. The sink's write strategy for filenames already existing in the bucket. Values: `create_new`, `replace`; `create_new` adds a new file with a suffix, while `replace` replaces the existent file. The legacy values `new` and `truncate` are still accepted.
  ---
  * {key}
  * String
  * Optional. Additional variables to be injected into the file template. See [file template](/classic/publish-data/sinks/s3-sink#file-template) for more details
{% /table %}

### Successful response example

```json
{
    "id": "t_6e8afdb8c691459b80e16541433f951b",
    "name": "p_test_0",
    "sql": "SELECT * FROM test",
    "description": null,
    "materialized": null,
    "cluster": null,
    "tags": {},
    "created_at": "2024-01-18 12:57:36.503843",
    "updated_at": "2024-01-19 09:27:12.069649",
    "version": 0,
    "project": null,
    "result": null,
    "ignore_sql_errors": false,
    "node_type": "sink",
    "dependencies": [
        "test"
    ],
    "params": [],
    "job": {
        "id": "685e7395-3b08-492b-9fe8-2944859d6a06",
        "kind": "sink",
        "status": "waiting",
        "created_at": "2024-01-19 15:58:46.688525",
        "updated_at": "2024-01-19 15:58:46.688532",
        "is_cancellable": true,
        "job_url": "https://api.split.tinybird.co/v0/jobs/685e7395-3b08-492b-9fe8-2944859d6a06",
        "pipe": {
            "id": "t_529f46626c324674b3a84cd820ac2649",
            "name": "p_test"
        }
    }
}
```

### Response codes

{% table %}
  * Code
  * Description
  ---
  * 200
  * OK
  ---
  * 404
  * Pipe, Node, or data connector not found
  ---
  * 403
  * Limit reached, Query includes forbidden keywords, Pipe is already a Sink Pipe
  ---
  * 400
  * Invalid or missing parameters, Pipe isn't a Sink Pipe
{% /table %}

## GET /v0/integrations/s3/policies/trust-policy

Retrieves the trust policy to attach to the IAM role used for the connection. External IDs differ for each Workspace but are shared between Branches of the same Workspace, so you don't need to change the trust policy for each Branch.

### Example

```shell
curl \
  -X GET "https://$TB_HOST/v0/integrations/s3/policies/trust-policy" \
  -H "Authorization: Bearer <ADMIN token>"

Successful response example
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "sts:AssumeRole",
            "Principal": {
                "AWS": "arn:aws:iam::123456789:root"
            },
            "Condition": {
                "StringEquals": {
                    "sts:ExternalId": "c6ee2795-aae3-4a55-a7a1-92d92fab0e41"
                }
            }
        }
    ]
}
```

### Response codes

{% table %}
  * Code
  * Description
  ---
  * 200
  * OK
  ---
  * 404
  * S3 integration not supported in your region
{% /table %}

## GET /v0/integrations/s3/policies/write-access-policy

Retrieves the trust policy to attach to the IAM role used for the connection. External IDs differ for each Workspace but are shared between Branches of the same Workspace, so you don't need to change the trust policy for each Branch.

### Example

```shell
curl \
  -X GET "https://$TB_HOST/v0/integrations/s3/policies/write-access-policy?bucket=test-bucket" \
  -H "Authorization: Bearer <ADMIN token>"
```

### Request parameters

{% table %}
  * Key
  * Type
  * Description
  ---
  * bucket
  * Optional[String]
  * Bucket to use for rendering the template. If not provided the '<bucket>' placeholder is used
{% /table %}

### Successful response example

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetBucketLocation",
                "s3:ListBucket"
            ],
            "Resource": "arn:aws:s3:::<bucket>"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject",
                "s3:PutObjectAcl"
            ],
            "Resource": "arn:aws:s3:::<bucket>/*"
        }
    ]
}
```

### Response codes

{% table %}
  * Code
  * Description
  ---
  * 200
  * OK
{% /table %}

## GET /v0/integrations/s3/settings

Retrieves the settings to attach to the IAM role used for the connection. External IDs differ for each Workspace but are shared between Branches of the same Workspace, so you don't need to generate specific IAM roles for each Branch.

### Example

```shell
curl \
  -X GET "https://$TB_HOST/v0/integrations/s3/settings" \
  -H "Authorization: Bearer <ADMIN token>"

Successful response example
{
    "principal": "arn:aws:iam::<aws_account_id>:root",
    "external_id": "<aws_external_id>"
}
```

### Response codes

{% table %}
  * Code
  * Description
  ---
  * 200
  * OK
  ---
  * 404
  * S3 integration not supported in your region
{% /table %}
