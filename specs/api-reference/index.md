---
last_updated: "2026-07-27T11:34:23.000Z"
title: API Overview
meta:
    description: Explore Tinybird APIs for ingesting data, managing Data Sources, Pipes, API Endpoints, Tokens, Jobs, Organizations, and ad hoc SQL queries.
---

# API overview

You can control Tinybird services using the API as an alternative to the UI and CLI.

The following APIs are available:

| API name | Description |
| -------- | -------- |
| [Analyze API](/api-reference/analyze-api) | Analyze a given NDJSON, CSV, or Parquet file to generate a Tinybird Data Source schema. |
| [Data Sources API](/api-reference/datasource-api) | List, create, update, or delete your Tinybird Data Sources, and insert or delete data from Data Sources. |
| [Events API](/api-reference/events-api) | Ingest NDJSON events with a simple HTTP POST request. |
| [Jobs API](/api-reference/jobs-api) | Get details on Tinybird jobs, and list the jobs for the last 48 hours or the last 100 jobs. |
| [Organizations API](/api-reference/organizations-api) | Manage organizations, members, workspaces, usage, and dedicated cluster operations. |
| [Pipes API](/api-reference/pipe-api) | Interact with your Pipes, including Pipes themselves, Endpoints, Materialized Views, and managing Copy jobs. |
| [Query API](/api-reference/query-api) | Query your Pipes and Data Sources inside Tinybird as if you were running SQL statements against a regular database. |
| [Environment Variables API](/api-reference/environment-variables-api) | Create, update, delete, and list variables that can be used in Pipes in a Workspace. |
| [Sink Pipes API](/api-reference/sink-pipes-api) | Create, delete, schedule, and trigger Sink Pipes. |
| [Tokens API](/api-reference/token-api) | List, create, update, or delete your Tinybird Static Tokens. |

Make all requests to Tinybird's Endpoints over TLS (HTTPS). All response bodies, including errors, are encoded as JSON.

{% callout type="tip" %}
You can get information on several Workspace operations by [monitoring your jobs](/forward/monitoring/jobs), using either the APIs or the built-in Tinybird Service Data Sources.
{% /callout %}

## Regions and endpoints

A workspace belongs to a region. The API for each region has a specific API base URL that you use to make API requests.

The following table lists the current regions and their corresponding API base URLs:

### Current Tinybird regions

{% table %}
* Region
* Provider
* Provider region
* API base URL
---
* Europe (London)
* GCP
* europe-west2
* [https://api.europe-west2.gcp.tinybird.co](https://api.europe-west2.gcp.tinybird.co)
---
* Europe (Frankfurt)
* GCP
* europe-west3
* [https://api.tinybird.co](https://api.tinybird.co)
---
* US East (Virginia)
* GCP
* us-east4
* [https://api.us-east.tinybird.co](https://api.us-east.tinybird.co)
---
* North America (Toronto)
* GCP
* northamerica-northeast2
* [https://api.northamerica-northeast2.gcp.tinybird.co](https://api.northamerica-northeast2.gcp.tinybird.co)
---
* Europe (Frankfurt)
* AWS
* eu-central-1
* [https://api.eu-central-1.aws.tinybird.co](https://api.eu-central-1.aws.tinybird.co)
---
* Europe (Ireland)
* AWS
* eu-west-1
* [https://api.eu-west-1.aws.tinybird.co](https://api.eu-west-1.aws.tinybird.co)
---
* US East (Virginia)
* AWS
* us-east-1
* [https://api.us-east.aws.tinybird.co](https://api.us-east.aws.tinybird.co)
---
* US West (Oregon)
* AWS
* us-west-2
* [https://api.us-west-2.aws.tinybird.co](https://api.us-west-2.aws.tinybird.co)
---
* Asia Pacific (Hong Kong)
* AWS
* ap-east-1
* [https://api.ap-east-1.aws.tinybird.co](https://api.ap-east-1.aws.tinybird.co)
---
* Asia Pacific (Sydney, Australia)
* AWS
* ap-southeast-2
* [https://api.ap-southeast-2.aws.tinybird.co](https://api.ap-southeast-2.aws.tinybird.co)
{% /table %}

## Tinybird Local

The Tinybird Local container exposes the following API endpoint:

- `http://localhost:7181/v0/`

See [Tinybird Local](/forward/core-concepts/tinybird-local) for more information.

## Authentication

{% $NAME %} uses Tokens for every API call. This ensures that each user or app can only access authorized data. See [Tokens](/forward/core-concepts/tokens).

You must make all API requests over HTTPS. Don't make calls over plain HTTP or send API requests without authentication.

{% snippet title="api-region-reminder" /%}

There are two ways to authenticate your requests in the {% $NAME %} API: using an authorization header, or using a URL parameter.

### Authorization header

You can send a Bearer authorization header to authenticate API calls. With cURL, use `-H "Authorization: Bearer <token>"`.

If you have a valid Token with read access to the particular Data Source, you can get a successful response by sending the following request:

```shell {% title="Authorization header Authenticated request" %}
curl \
-X GET \
-H "Authorization: Bearer <token>" \
"{% user("apiHost") %}/v0/sql?q=SELECT+*+FROM+<pipe>"
```

### URL parameter

You can also specify the Token using a parameter in the URL, using `token=<token>`. For example:

```shell {% title="URL parameter authenticated request" %}
curl -X GET \
"{% user("apiHost") %}/v0/sql?q=SELECT+*+FROM+<pipe>&token=<token>"
```

## Compression

To compress API responses, add `Accept-Encoding: gzip` to your requests. For example:

```shell {% title="Request with compressed response" %}
curl \
-X GET \
-H "Authorization: Bearer <token>" \
-H "Accept-Encoding: gzip" \
"{% user("apiHost") %}/v0/sql?q=SELECT+*+FROM+<pipe>"
```

## Errors

{% $NAME %}'s API returns standard HTTP success or error status codes. In case of errors, responses include additional information in JSON format.

The following table lists the error status codes.

### Response codes

{% table %}
* Code {% width="15%" %}
* Description
---
* 400
* Bad request. This could be due to a missing parameter in a request, for instance
---
* 403
* Forbidden. Provided auth token doesn't have the right scope or the Data Source isn't available
---
* 404
* Not found
---
* 405
* HTTP Method not allowed
---
* 408
* Request timeout (for example, query execution exceeded the time limit)
---
* 409
* You need to resubmit the request due to a conflict with the current state of the target source (e.g.: you need to delete a Materialized View)
---
* 411
* No valid Content-Length header containing the length of the message-body
---
* 413
* The message body is too large
---
* 429
* Too many requests. When over the rate limits of your account
---
* 500
* Unexpected error
{% /table %}

## Limits

{% snippet title="forward-limits-reminder" /%}

## Organizations API

You can retrieve information about the members of your organization using the following endpoint:

```shell {% title="Get organization members" %}
curl -X GET \
  -H "Authorization: Bearer <token>" \
  "{% user("apiHost") %}/v0/organizations/<organization_id>/members"
```

Replace `<organization_id>` with your organization ID. You can find your organization ID in the Tinybird UI under **Organization settings**.

For dedicated infrastructure cluster management endpoints, see the [Organizations API reference](/api-reference/organizations-api).

## Versioning

All {% $NAME %} APIs are versioned with a version string specified in the base URL. Always use the latest API available.

When versioning services, Tinybird adheres to [semantic versioning](https://semver.org/) rules.

## Reserved words

The following keywords are reserved. You can't use them to name Data Sources, Pipes, nodes or Workspaces. Case is ignored.

- `Array`
- `Bool`
- `Boolean`
- `Date`
- `Date32`
- `DateTime`
- `DateTime32`
- `DateTime64`
- `Decimal`
- `Decimal128`
- `Decimal256`
- `Decimal32`
- `Decimal64`
- `Enum`
- `Enum16`
- `Enum8`
- `FixedString`
- `Float32`
- `Float64`
- `IPv4`
- `IPv6`
- `Int128`
- `Int16`
- `Int256`
- `Int32`
- `Int64`
- `Int8`
- `MultiPolygon`
- `Point`
- `Polygon`
- `Ring`
- `String`
- `TABLE`
- `UInt128`
- `UInt16`
- `UInt256`
- `UInt32`
- `UInt64`
- `UInt8`
- `UUID`
- `_temporary_and_external_tables`
- `add`
- `after`
- `all`
- `and`
- `anti`
- `any`
- `array`
- `as`
- `asc`
- `asof`
- `between`
- `by`
- `case`
- `collate`
- `column`
- `columns`
- `cross`
- `cube`
- `custom_error`
- `day_diff`
- `default`
- `defined`
- `desc`
- `distinct`
- `else`
- `end`
- `enumerate_with_last`
- `error`
- `exists`
- `from`
- `full`
- `functions`
- `generateRandom`
- `global`
- `group`
- `having`
- `if`
- `ilike`
- `in`
- `inner`
- `insert`
- `interval`
- `into`
- `join`
- `left`
- `like`
- `limit`
- `limits`
- `max`
- `min`
- `not`
- `null`
- `numbers_mt`
- `on`
- `one`
- `or`
- `order`
- `outer`
- `prewhere`
- `public`
- `right`
- `sample`
- `select`
- `semi`
- `split_to_array`
- `sql_and`
- `sql_unescape`
- `system`
- `table`
- `then`
- `tinybird`
- `to`
- `union`
- `using`
- `where`
- `with`
- `zeros_mt`
- `zip`

Pipe, Data Source and node names are globally unique. You can't use an alias for a column that matches a globally unique name.
