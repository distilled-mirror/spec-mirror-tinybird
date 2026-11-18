---
last_updated: "2026-11-04T12:15:44.000Z"
title: Events API Reference
meta:
    description: With the Tinybird Events API you can ingest thousands of JSON events per second.
headingMaxLevels: 2
---

# Events API

{% snippet title="api-region-reminder" /%}

Use the Events API to ingest JSON events with an HTTP POST request. See [Events API](/forward/ingest-data/events-api) and [Ingestion limits](/forward/pricing/limits#ingestion-limits).

All endpoints require authentication using a Token with `DATASOURCES:APPEND` or `DATASOURCES:CREATE` scope.

## POST /v0/events

Use this endpoint to send NDJSON (new-line delimited JSON) events to a Data Source.

### Request parameters

{% table %}
  * Key {% width="10%" %}
  * Type {% width="10%" %}
  * Description
  ---
  * name
  * String
  * name or ID of the target Data Source to append data to it
  ---
  * format
  * String
  * `ndjson` by default. Set to `json` to send a single JSON message. Other values return a 400 error.
  ---
  * wait
  * Bool
  * `false` by default. Set to `true` to wait until the database acknowledges the write. Use this flag to retry database errors. This setting adds latency, so turn it on when avoiding data loss is critical. Leave it off otherwise.
{% /table %}

### Returned HTTP status codes

{% table %}
* Status Code {% width="15%" %}
* Description
---
* 200
* The data has been inserted into the database. The write has been acknowledged. The request 'wait' parameter was enabled.
---
* 202
* Tinybird has processed the data and sends it to the database asynchronously. The database hasn't acknowledged the write yet. The request `wait` parameter was off.
---
* 400
* The request is invalid. The body contains more information. A common cause is a missing `name` parameter. Tinybird hasn't inserted any data, and you shouldn't retry the request.
---
* 403
* The token isn't valid. The request shouldn't be retried.
---
* 404
* The token's Workspace doesn't belong to this cluster, or the Data Source doesn't exist in the Workspace. The Workspace is probably removed or in another cluster. The request shouldn't be retried, ensure the token's region and the Tinybird domain matches, and that the Data Source is deployed.
---
* 413
* The request payload size exceeds the limit for your plan. For example, the Free plan limit is 10 MB. Retry the request with a smaller payload.
---
* 422
* An error in a Materialized View interrupted ingestion. Retrying may duplicate data, but not retrying may lose data. Don't retry. Review attached Materialized Views and contact support if the issue persists.
---
* 429
* The request-per-second limit has been reached. The default limit is 1000 requests per second per Data Source. Contact support for increased capacity. Retry after a delay, using exponential backoff and a limited number of retries.
---
* 500
* An unexpected error occurred. The body contains more information. Retry the request and contact support if the issue persists.
---
* 503
* The service is temporarily unavailable. Increased payload size, network issues, or Tinybird infrastructure issues can cause throughput to process more slowly than expected. Tinybird hasn't inserted any data, so it's safe to retry. Contact support if the issue persists.
---
* 0x07 GOAWAY
* HTTP2 only. Too many alive connections. Recreate the connection and retry.
{% /table %}

### Compression

You can compress JSON events with Gzip or Zstandard and send the compressed payload to the Events API. You must include the `Content-Encoding` header in the request, set to `gzip` or `zstd` to indicate the compression algorithm used.

## Idempotency

The Events API isn't idempotent. Sending the same data more than once inserts it multiple times.

The following responses are safe to retry without risk of duplication:

- **HTTP 429 (rate-limited):** Tinybird didn't process the original request.
- **HTTP 503 (service unavailable):** No data was inserted.

For other transient failures, [Tinybird handles retries internally](/forward/ingest-data/ingestion-protection): partially inserted data is deduplicated for up to 5 hours.

## Examples

### NDJSON messages

The following example shows how to push single NDJSON messages using the Events API:

```shell {% title="Send individual NDJSON messages" %}
curl \
    -H "Authorization: Bearer <import_token>" \
    -d '{"date": "2020-04-05 00:05:38", "city": "Chicago"}' \
    '{% user("apiHost") %}/v0/events?name=events_test'
```

### JSON messages

The following example shows how to push single JSON messages using the Events API:


```shell {% title="Send individual JSON messages" %}
curl \
    -H "Authorization: Bearer <import_token>" \
    -d $'{ \
          "date": "2020-04-05 00:05:38", \
          "city": "Chicago" \
        }' \
    '{% user("apiHost") %}/v0/events?name=events_test&format=json'
```

### Multiple NDJSON messages

The following example shows how to push multiple NDJSON messages using the Events API. Notice the '$' before the JSON events. It's needed in order for Bash to replace the '\\n'. curl doesn't do it automatically.

```shell {% title="Send many NDJSON events." %}
curl \
    -H "Authorization: Bearer <import_token>" \
    -d $'{"date": "2020-04-05 00:05:38", "city": "Chicago"}\n{"date": "2020-04-05 00:07:22", "city": "Madrid"}\n' \
    '{% user("apiHost") %}/v0/events?name=events_test'
```

### Gzip compressed payload

The following example shows how to push a Gzip compressed payload using the Events API, where 'body.gz' is a batch of NDJSON events.

```shell {% title="Send a Gzip compressed payload." %}
curl \
    -H "Authorization: Bearer <import_token>" \
    -H "Content-Encoding: gzip" \
    --data-binary @body.gz \
    '{% user("apiHost") %}/v0/events?name=events_example'
```

### Zstandard compressed payload

The following example shows how to push Zstandard compressed NDJSON events from a file named `body.ndjson.zst` using the Events API.

```shell {% title="Send a ZSTD compressed payload." %}
curl \
    -H "Authorization: Bearer <import_token>" \
    -H "Content-Encoding: zstd" \
    --data-binary @body.ndjson.zst \
    '{% user("apiHost") %}/v0/events?name=events_example'
```
