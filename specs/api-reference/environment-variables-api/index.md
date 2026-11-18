---
last_updated: "2026-11-04T12:15:44.000Z"
title: Environment Variables API
meta:
    description: The Environment Variables API allows you to create, update, delete and list environment variables that can be used in Pipes in a Workspace.
headingMaxLevels: 2
---

# Environment variables API

{% snippet title="api-region-reminder" /%}

Use the Environment Variables API to create, update, delete, and list environment variables that you can use in Pipes in a Workspace. 

Environment variables allow you to store sensitive information, such as access secrets and host names, in your Workspace. Environment variables are encrypted at rest.

{% callout type="info" %}
Using the Environment Variables API requires a Workspace admin token.
{% /callout %}

## Environment variable types {% id="environment-variables-types" %}

The Environment Variables API supports different types of environment variables:

| Environment variable type | Comments |
| ------------------------- | -------- |
| `secret`                  | Used to store passwords and other secrets, automatically prevents Endpoint from exposing its value. It's the default type. |

## Limits

{% snippet title="forward-limits-reminder" /%}

The Environment Variables API has the following limits:

- 5 requests per second.
- 100 environment variables per Workspace.
- 8 KB max size of the `value` attribute.

## Templating

After creating environment variables in a Workspace, you can use the `tb_secret` template function to replace the original value:

```tinybird
%
SELECT
    *
FROM postgresql('host:post', 'database', 'table', 'user', {{tb_secret('pg_password')}})
```

Environment variables values are rendered as `String` data type. If you need to use a different type, use any of the [functions to cast a String value to a given type](/sql-reference/functions). For example:

```tinybird
%
SELECT
    *
FROM table
WHERE int_value = toUInt8({{tb_secret('int_secret')}})
```

## Staging and production use

If you have staging and production Workspaces, create the same environment variables with the same name in both Workspaces, changing only their corresponding value.

{% callout type="tip" %}
Tinybird doesn't allow you to create an API Endpoint when exposing environment variables with `type=secret` in a SELECT clause. So, while it's possible to have a node that uses the logic `SELECT {{tb_secret('username')}}`, you can't publish that node as a Copy Pipe or API Endpoint.
{% /callout %}

## Branch use

In Tinybird Forward, you can create, update, and delete environment variables in Branches, but not in Releases. In Tinybird Classic, you can use environment variables in Branches, but you must create them in the main Workspace initially. Environment variables have the same value in the main Workspace as in the Branches. You can't create a environment variable in a Classic Branch to be deployed in the main Workspace.

## POST /v0/variables/?

Creates a new environment variable.

### Restrictions

Environment variables names are unique for a Workspace.

### Example

```shell
curl \
  -X POST "https://$TB_HOST/v0/variables" \
  -H "Authorization: Bearer <ADMIN token>" \
  -d "type=secret" \
  -d "name=test_password" \
  -d "value=test"
```

### Request parameters

{% table %}
  * Key
  * Type
  * Description
  ---
  * type
  * String (optional)
  * The type of the variable. Defaults to `secret`
  ---
  * name
  * String
  * The name of the variable
  ---
  * value
  * String
  * The variable value
{% /table %}

### Successful response example

```json
{
    "name": "test_token",
    "created_at": "2024-06-21T10:27:57",
    "updated_at": "2024-06-21T10:27:57",
    "edited_by": "token: 'admin token'"
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
  * 400
  * Invalid or missing parameters, or the 100 variables limit per Workspace is reached
  ---
  * 403
  * Invalid token, or the variable is created in a Classic Branch or a Forward Release
  ---
  * 404
  * Workspace not found
  ---
  * 429
  * Rate limit of 5 requests per second reached
{% /table %}

## DELETE /v0/variables/(.+)

Deletes an environment variable.

### Example

```shell
curl \
  -X DELETE "https://$TB_HOST/v0/variables/test_password" \
  -H "Authorization: Bearer <ADMIN token>"
```

### Successful response example

```json
{
    "ok": true
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
  * 400
  * Invalid or missing parameters
  ---
  * 403
  * Invalid token, or the variable is changed in a Classic Branch or a Forward Release
  ---
  * 404
  * Workspace or variable not found
{% /table %}

## PUT /v0/variables/(.+)

Updates an environment variable.

### Example

```shell
curl \
  -X PUT "https://$TB_HOST/v0/variables/test_password" \
  -H "Authorization: Bearer <ADMIN token>" \
  -d "value=new_value"
```

### Successful response example

```json
{
    "name": "test_password",
    "type": "secret",
    "created_at": "2024-06-21T10:27:57",
    "updated_at": "2024-06-21T10:29:57",
    "edited_by": "token: 'admin token'"
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
  * 400
  * Invalid or missing parameters
  ---
  * 403
  * Invalid token, or the variable is changed in a Classic Branch or a Forward Release
  ---
  * 404
  * Workspace or variable not found
{% /table %}

## GET /v0/variables/?

Retrieves all Workspace environment variables. The value isn't returned.

### Example

```shell
curl \
  -X GET "https://$TB_HOST/v0/variables" \
  -H "Authorization: Bearer <ADMIN token>"
```

### Successful response example

```json
{
    "variables": [
        {
            "name": "test_token",
            "type": "secret",
            "created_at": "2024-06-21T10:27:57",
            "updated_at": "2024-06-21T10:27:57",
            "edited_by": "token: 'admin token'"
        },
        {
            "name": "test_token2",
            "type": "secret",
            "created_at": "2024-06-21T10:27:57",
            "updated_at": "2024-06-21T10:29:57",
            "edited_by": "token: 'admin token'"
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
  * 400
  * Invalid or missing parameters
  ---
  * 403
  * Limit reached or token invalid
  ---
  * 404
  * Workspace not found
{% /table %}

## GET /v0/variables/(.+)

Fetches information about a particular environment variable. The value isn't returned.

### Example

```shell
curl \
  -X GET "https://$TB_HOST/v0/variables/test_password" \
  -H "Authorization: Bearer <ADMIN token>"
```

### Successful response example

```json
{
    "name": "test_password",
    "type": "secret",
    "created_at": "2024-06-21T10:27:57",
    "updated_at": "2024-06-21T10:27:57",
    "edited_by": "token: 'admin token'"
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
  * 400
  * Invalid or missing parameters
  ---
  * 403
  * Limit reached or token invalid
  ---
  * 404
  * Workspace or variable not found
{% /table %}
