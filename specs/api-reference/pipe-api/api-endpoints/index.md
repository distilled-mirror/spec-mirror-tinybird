---
last_updated: "2026-09-22T11:50:48.000Z"
title: Pipes API endpoints reference
meta:
    description: The Pipes API helps you manage your Pipes. Use the API Endpoints service to publish or unpublish your Pipes as Endpoints.
headingMaxLevels: 2
---

# Pipes API endpoint service

{% snippet title="api-region-reminder" /%}

{% snippet title="pipes-api-snippet" /%}

{% callout type="caution" %}
Adding or modifying pipes or data sources in [Tinybird Local](/forward/core-concepts/tinybird-local) and [Tinybird Forward](/forward) can only be done through deployments.
{% /callout %}

## Endpoints

## POST /v0/pipes/(.+)/nodes/(.+)/endpoint

Publishes an API endpoint

**Publishing an endpoint**

```bash
curl -X POST \
    -H "Authorization: Bearer <PIPE:CREATE token>" \
    "https://<your_host>/v0/pipes/:pipe/nodes/:node/endpoint"
```

**Successful response**

```json
{
    "id": "t_60d8f84ce5d349b28160013ce99758c7",
    "name": "my_pipe",
    "description": "this is my pipe description",
    "nodes": [{
        "id": "t_bd1e095da943494d9410a812b24cea81",
        "name": "get_all",
        "sql": "SELECT * FROM my_datasource",
        "description": "This is a description for the **first** node",
        "materialized": null,
        "dependencies": ["my_datasource"],
        "tags": {},
        "created_at": "2019-09-03 19:56:03.704840",
        "updated_at": "2019-09-04 07:05:53.191437",
        "version": 0,
        "project": null,
        "result": null,
        "ignore_sql_errors": false
    }],
    "endpoint": "t_bd1e095da943494d9410a812b24cea81",
    "created_at": "2019-09-03 19:56:03.193446",
    "updated_at": "2019-09-10 07:18:39.797083",
    "parent": null
}
```

{% callout %}
The response will contain a `token` if there’s a **unique READ token** for this pipe. You could use this token to share your endpoint.
{% /callout %}

### Response codes

{% table %}
  * Code
  * Description
  ---
  * 200
  * No error
  ---
  * 400
  * Wrong node id
  ---
  * 403
  * Forbidden. Provided token doesn’t have permissions to publish a pipe, it needs `ADMIN` or `PIPE:CREATE`
  ---
  * 404
  * Pipe not found
{% /table %}

## DELETE /v0/pipes/(.+)/nodes/(.+)/endpoint

Unpublishes an API endpoint

**Unpublishing an endpoint**

```bash
curl -X DELETE \
    -H "Authorization: Bearer <PIPE:CREATE token>" \
    "https://<your_host>/v0/pipes/:pipe/nodes/:node/endpoint"
```

### Response codes

{% table %}
  * Code
  * Description
  ---
  * 200
  * No error
  ---
  * 400
  * Wrong node id
  ---
  * 403
  * Forbidden. Provided token doesn’t have permissions to publish a pipe, it needs `ADMIN` or `PIPE:CREATE`
  ---
  * 404
  * Pipe not found
{% /table %}

## Response formats

Endpoints support multiple response formats, which can be specified by appending the format extension to the endpoint URL:

- `.json` - Returns data in JSON format (default)
- `.csv` - Returns data in CSV format, including a header row with column names by default
- `.ndjson` - Returns data as newline-delimited JSON
- `.parquet` - Returns data in Parquet format
- `.prometheus` - Returns data in Prometheus format (requires specific data structure)

```shell {% title="Examples of endpoints with different formats" %}
{% user("apiHost") %}/v0/pipes/your_pipe_name.json
{% user("apiHost") %}/v0/pipes/your_pipe_name.csv
{% user("apiHost") %}/v0/pipes/your_pipe_name.ndjson
{% user("apiHost") %}/v0/pipes/your_pipe_name.parquet
{% user("apiHost") %}/v0/pipes/your_pipe_name.prometheus
```

## Omit the CSV header row

The `csv` format includes a header row with column names by default, matching ClickHouse®'s `CSVWithNames` output. Pass `__tb__csv_header=false` to omit that row and get plain `CSV` instead.

- Only valid together with the `csv` format. Using it with any other format returns a `400` error.
- Accepts `true`, `false`, `1`, or `0`. Defaults to true. Any other value returns a `400` error.

```shell {% title="Request a headerless CSV response" %}
{% user("apiHost") %}/v0/pipes/your_pipe_name.csv?__tb__csv_header=false
```

The response's `Content-Type` header reflects the choice, with `header=present` or `header=absent`, so a client can confirm which shape it received without parsing the body.

### Concatenate paginated CSV exports

Request the header on the first page only, then omit it on every continuation page. The pages concatenate into a single valid CSV file, even when combined as raw gzip data without decompressing each page first.

This example paginates with a keyset (cursor) rather than `LIMIT`/`OFFSET`. For large exports, prefer a keyset: `OFFSET` has to scan and discard every row ahead of it, so the cost grows with page depth, while a keyset filters on the last row's own sort key and costs the same on every page.

```shell {% title="Page 1: keep the header" %}
curl -s -H "Accept-Encoding: gzip" \
  "{% user("apiHost") %}/v0/pipes/your_pipe_name.csv?page_size=1000&token=<PIPE:READ token>" \
  --output page1.csv.gz
```

```shell {% title="Page 2: omit the header, resume from the last row" %}
curl -s -H "Accept-Encoding: gzip" \
  "{% user("apiHost") %}/v0/pipes/your_pipe_name.csv?page_size=1000&cursor_ts=<last_ts>&cursor_hash=<last_hash>&__tb__csv_header=false&token=<PIPE:READ token>" \
  --output page2.csv.gz
```

```shell {% title="Concatenate and decompress" %}
cat page1.csv.gz page2.csv.gz > combined.csv.gz
gunzip -c combined.csv.gz
```

The combined file contains exactly one header row, at the top. `cursor_ts` and `cursor_hash` are the pipe's own parameters, not Tinybird's: pick a unique, strictly ordered tiebreaker for your data (for example a timestamp plus a hash or ID) so the cursor advances consistently, including across timestamp ties.

### CSV headers on the Query API

`__tb__csv_header` applies only to `/v0/pipes/<pipe>.csv`, where the `.csv` extension always maps to `CSVWithNames`. The [Query API](/forward/query-data/sql-api#available-response-formats) doesn't need this parameter: choose `FORMAT CSV` or `FORMAT CSVWithNames` directly in the query text instead.
