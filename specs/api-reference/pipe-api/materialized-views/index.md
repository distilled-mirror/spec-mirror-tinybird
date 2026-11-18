---
last_updated: "2026-09-22T11:50:48.000Z"
title: Pipes API Materialized Views and Populates reference
seo_title: "Materialized Views and Populates Reference"
meta:
    description: The Pipes API enables you to manage your Pipes. Use the Materialized Views and Populates service to create, delete, or populate Materialized Views.
headingMaxLevels: 2
---

# Pipes API Materialized Views and Populates service

{% snippet title="api-region-reminder" /%}

{% snippet title="pipes-api-snippet" /%}

{% callout type="caution" %}
Adding or modifying pipes or data sources in [Tinybird Local](/forward/core-concepts/tinybird-local) and [Tinybird Forward](/forward) can only be done through deployments.
{% /callout %}

## Materialized Views and populates

{% callout type="caution" %}
The population endpoint in this section is Classic-only. In [Tinybird Local](/forward/core-concepts/tinybird-local) and [Tinybird Forward](/forward), calling it returns a 400 error: `Forward workspaces do not support populating materialized views on-demand. Data Sources are automatically backfilled during deployments.` See [populating Materialized Views](/forward/core-concepts/materialized-views#populating-materialized-views) for how backfilling works in Forward.
{% /callout %}

## POST /v0/pipes/(.+)/nodes/(.+)/population

Populates a Materialized View

**Populating a Materialized View**

```bash
curl
    -H "Authorization: Bearer <PIPE:CREATE token>" \
    -X POST "https://<your_host>/v0/pipes/:pipe/nodes/:node/population" \
    -d "populate_condition=toYYYYMM(date) = 202203"
```

The response will not be the final result of the import but a Job. You can check the job status and progress using the [Jobs API](/docs/api-reference/jobs-api).

Alternatively you can use a query like this to check the operations related to the populate Job:

**Check populate jobs in the datasources_ops_log including dependent Materialized Views triggered**

```sql
SELECT *
FROM tinybird.datasources_ops_log
WHERE
    timestamp > now() - INTERVAL 1 DAY
    AND operation_id IN (
        SELECT operation_id
        FROM tinybird.datasources_ops_log
        WHERE
            timestamp > now() - INTERVAL 1 DAY
            and datasource_id = '{the_datasource_id}'
            and job_id = '{the_job_id}'
    )
ORDER BY timestamp ASC
```

When a populate job fails for the first time, the Materialized View is automatically unlinked. In that case you can get failed population jobs and their errors to fix them with a query like this:

**Check failed populate jobs**

```sql
SELECT *
FROM tinybird.datasources_ops_log
WHERE
    datasource_id = '{the_datasource_id}'
    AND pipe_name = '{the_pipe_name}'
    AND event_type LIKE 'populateview%'
    AND result = 'error'
ORDER BY timestamp ASC
```

Alternatively you can use the `unlink_on_populate_error='true'` flag to always unlink the Materialized View if the populate job does not work as expected.

### Request parameters

{% table %}
  * Key
  * Type
  * Description
  ---
  * token
  * String
  * Auth token. Ensure it has the `PIPE:CREATE` scope on it
  ---
  * populate_subset
  * Float
  * Optional. Populate with a subset percent of the data (limited to a maximum of 2M rows), this is useful to quickly test a materialized node with some data. The subset must be greater than 0 and lower than 0.1. A subset of 0.1 means a 10 percent of the data in the source Data Source will be used to populate the Materialized View. It has precedence over `populate_condition`
  ---
  * populate_condition
  * String
  * Optional. Populate with a SQL condition to be applied to the trigger Data Source of the Materialized View. For instance, `populate_condition='date == toYYYYMM(now())'` it’ll populate taking all the rows from the trigger Data Source which `date` is the current month. `populate_condition` is not taken into account if the `populate_subset` param is present. Including in the `populate_condition` any column present in the Data Source `engine_sorting_key` will make the populate job process less data.
  ---
  * truncate
  * String
  * Optional. Default is `false`. Populates over existing data, useful to populate past data while new data is being ingested. Use `true` to truncate the Data Source before populating.
  ---
  * unlink_on_populate_error
  * String
  * Optional. Default is `false`. If the populate job fails the Materialized View is unlinked and new data won’t be ingested in the Materialized View.
{% /table %}

### Response codes

{% table %}
  * Code
  * Description
  ---
  * 200
  * No error
  ---
  * 400
  * Node is not materialized
  ---
  * 403
  * Forbidden. Provided token doesn’t have permissions to append a node to the pipe, it needs `ADMIN` or `PIPE:CREATE`
  ---
  * 404
  * Pipe not found, Node not found
{% /table %}

## POST /v0/pipes/(.+)/nodes/(.+)/materialization

Creates a Materialized View

**Creating a Materialized View**

```bash
curl \
    -H "Authorization: Bearer <PIPE:CREATE token>" \
    -X POST "https://<your_host>/v0/pipes/:pipe/nodes/:node/materialization?datasource=my_data_source_name&populate=true"
```

### Request parameters

{% table %}
  * Key
  * Type
  * Description
  ---
  * token
  * String
  * Auth token. Ensure it has the `PIPE:CREATE` scope on it
  ---
  * datasource
  * String
  * Required. Specifies the name of the destination Data Source where the Materialized View schema is defined. If the Data Source does not exist, it creates automatically with the default settings.
  ---
  * override_datasource
  * Boolean
  * Optional. Default `false` When the target Data Source of the Materialized View exists in the Workspace it’ll be overridden by the `datasource` specified in the request.
  ---
  * populate
  * Boolean
  * Optional. Default `false`. When `true`, a job is triggered to populate the destination datasource.
  ---
  * populate_subset
  * Float
  * Optional. Populate with a subset percent of the data (limited to a maximum of 2M rows), this is useful to quickly test a materialized node with some data. The subset must be greater than 0 and lower than 0.1. A subset of 0.1 means a 10 percent of the data in the source Data Source will be used to populate the Materialized View. Use it together with `populate=true`, it has precedence over `populate_condition`
  ---
  * populate_condition
  * String
  * Optional. Populate with a SQL condition to be applied to the trigger Data Source of the Materialized View. For instance, `populate_condition='date == toYYYYMM(now())'` it’ll populate taking all the rows from the trigger Data Source which `date` is the current month. Use it together with `populate=true`. `populate_condition` is not taken into account if the `populate_subset` param is present. Including in the `populate_condition` any column present in the Data Source `engine_sorting_key` will make the populate job process less data.
  ---
  * unlink_on_populate_error
  * String
  * Optional. Default is `false`. If the populate job fails the Materialized View is unlinked and new data won’t be ingested in the Materialized View.
  ---
  * engine
  * String
  * Optional. Engine for destination Materialized View. If the Data Source already exists, the settings are not overridden.
  ---
  * engine_*
  * String
  * Optional. Engine parameters and options. Requires the `engine` parameter. If the Data Source already exists, the settings are not overridden. [Check Engine Parameters and Options for more details](/docs/api-reference/datasource-api)
{% /table %}

SQL query for the materialized node must be sent in the body encoded in utf-8

### Response codes

{% table %}
  * Code
  * Description
  ---
  * 200
  * No error
  ---
  * 400
  * Node already being materialized
  ---
  * 403
  * Forbidden. Provided token doesn’t have permissions to append a node to the pipe, it needs `ADMIN` or `PIPE:CREATE`
  ---
  * 404
  * Pipe not found, Node not found
  ---
  * 409
  * The Materialized View already exists or `override_datasource` cannot be performed
{% /table %}

## DELETE /v0/pipes/(.+)/nodes/(.+)/materialization

Removes a Materialized View

By removing a Materialized View, nor the Data Source nor the Node are deleted. The Data Source will still be present, but will stop receiving data from the Node.

**Removing a Materialized View**

```bash
curl                 -H "Authorization: Bearer <PIPE:CREATE token>" \
    -X DELETE "https://<your_host>/v0/pipes/:pipe/nodes/:node/materialization"
```

### Response codes

{% table %}
  * Code
  * Description
  ---
  * 204
  * No error, Materialized View removed
  ---
  * 403
  * Forbidden. Provided token doesn’t have permissions to append a node to the pipe, it needs `ADMIN` or `PIPE:CREATE`
  ---
  * 404
  * Pipe not found, Node not found
{% /table %}
