---
last_updated: "2026-11-04T12:15:44.000Z"
title: Token API Reference
meta:
    description: In order to read, append or import data into you Tinybird account, you'll need a Token with the right permissions.
headingMaxLevels: 2
---

# Token API

{% snippet title="api-region-reminder" /%}

Use the Token API to list, create, update, or delete your Tinybird Static Tokens.

{% callout %}
New to Static Tokens? Read more about them [in the Tokens docs](/forward/core-concepts/tokens).
{% /callout %}
{% callout type="caution" %}
Tinybird Forward doesn't support the `:sql_filter` suffix on resource-scoped Tokens, such as `DATASOURCES:READ:datasource_name:sql_filter` and `PIPES:READ:pipe_name:sql_filter`. Using this suffix results in an error.
{% /callout %}
{% callout type="caution" %}
In Tinybird Forward, creating, updating, or deleting Tokens with `DATASOURCES:READ`, `DATASOURCES:APPEND`, or `PIPES:READ` scopes returns a `403` error. Manage these Tokens in your datafiles and deployments.
{% /callout %}

All endpoints require authentication using a Token with `TOKENS` or `ADMIN` scope.

## GET /v0/tokens/?

Retrieves all workspace Static Tokens.

**Get all tokens**

```bash
curl -X GET \
    -H "Authorization: Bearer <ADMIN token>" \
    "https://<your_host>/v0/tokens"
```

A list of your Static Tokens and their scopes will be sent in the response.

**Successful response**

```json
{
    "tokens": [
        {
            "name": "admin token",
            "description": "",
            "scopes": [
                { "type": "ADMIN" }
            ],
            "token": "p.token"
        },
        {
            "name": "import token",
            "description": "",
            "scopes": [
                { "type": "DATASOURCES:CREATE" }
            ],
            "token": "p.token0"
        },
        {
            "name": "token name 1",
            "description": "",
            "scopes": [
                { "type": "DATASOURCES:READ", "resource": "table_name_1" },
                { "type": "DATASOURCES:APPEND", "resource": "table_name_1" }
            ],
            "token": "p.token1"
        },
        {
            "name": "token name 2",
            "description": "",
            "scopes": [
                { "type": "PIPES:READ", "resource": "pipe_name_2" }
            ],
            "token": "p.token2"
        }
    ]
}
```

## POST /v0/tokens/?

Creates a new Token: Static or JWT

**Creating a new Static Token**

```bash
curl -X POST \
    -H "Authorization: Bearer <ADMIN token>" \
    "https://<your_host>/v0/tokens/" \
    -d "name=test&scope=DATASOURCES:APPEND:table_name&scope=DATASOURCES:READ:table_name"
```

### Request parameters

{% table %}
  * Key
  * Type
  * Description
  ---
  * name
  * String
  * Name of the token
  ---
  * description
  * String
  * Optional. Markdown text with a description of the token.
  ---
  * scope
  * String
  * Scope(s) to set. Format is [SCOPE:TYPE[:arg][:filter]](#id2). This is only used for the Static Tokens
{% /table %}

**Successful response**

```json
{
    "name": "token_name",
    "description": "",
    "scopes": [
        { "type": "DATASOURCES:APPEND", "resource": "table_name" }
        { "type": "DATASOURCES:READ", "resource": "table_name", "filter": "department = 1"},
    ],
    "token": "p.token"
}
```

When creating a token with `filter` whenever you use the token to read the table, it will be filtered. For example, if table is `events_table` and `filter` is `date > '2018-01-01' and type == 'foo'` a query like `select count(1) from events_table` will become `select count(1) from events_table where date > '2018-01-01' and type == 'foo'`

**Creating a new token with filter**

```bash
curl -X POST \
    -H "Authorization: Bearer <ADMIN token>" \
    "https://<your_host>/v0/tokens/" \
    -d "name=test&scope=DATASOURCES:READ:table_name:column==1"
```

If we provide an `expiration_time` in the URL, the token will be created as a JWT Token.

**Creating a new JWT Token**

```bash
curl -X POST \
    -H "Authorization: Bearer <ADMIN token>" \
    "https://<your_host>/v0/tokens?name=jwt_token&expiration_time=1710000000" \
    -d '{"scopes": [{"type": "PIPES:READ", "resource": "requests_per_day", "fixed_params": {"user_id": 3}}]}'
```

{% callout %}
In multi-tenant applications, you can use this endpoint to create a JWT token for a specific tenant where each user has their own token with a fixed set of scopes and parameters
{% /callout %}

## POST /v0/tokens/(.+)/refresh

Refresh the Static Token without modifying name, scopes or any other attribute. Specially useful when a Token is leaked, or when you need to rotate a Token. A member token can always refresh itself, even with the Viewer role.

**Refreshing a Static Token**

```bash
curl -X POST \
    -H "Authorization: Bearer <ADMIN token>" \
    "https://<your_host>/v0/tokens/:token_name/refresh"
```

When successfully refreshing a token, new information will be sent in the response

**Successful response**

```json
{
    "name": "token name",
    "description": "",
    "scopes": [
        { "type": "DATASOURCES:READ", "resource": "table_name" }
    ],
    "token": "NEW_TOKEN"
}
```

### Request parameters

{% table %}
  * Key
  * Type
  * Description
  ---
  * auth_token
  * String
  * Token. Ensure it has the `TOKENS` scope on it
{% /table %}

### Response codes

{% table %}
  * Code
  * Description
  ---
  * 200
  * No error
  ---
  * 403
  * Forbidden. Provided token doesn’t have permissions to drop the token. A token is not allowed to remove itself, it needs `ADMIN` or `TOKENS` scope
{% /table %}

## GET /v0/tokens/(.+)

Fetches information about a particular Static Token.

**Getting token info**

```bash
curl -X GET \
    -H "Authorization: Bearer <ADMIN token>" \
    "https://<your_host>/v0/tokens/:token"
```

Returns a json with name and scopes.

**Successful response**

```json
{
    "name": "token name",
    "description": "",
    "scopes": [
        { "type": "DATASOURCES:READ", "resource": "table_name" }
    ],
    "token": "p.TOKEN"
}
```

## DELETE /v0/tokens/(.+)

Deletes a Static Token .

**Deleting a token**

```bash
curl -X DELETE \
    -H "Authorization: Bearer <ADMIN token>" \
    "https://<your_host>/v0/tokens/:token"
```

## PUT /v0/tokens/(.+)

Modifies a Static Token. More than one scope can be sent per request, all of them will be added as Token scopes. Every time a Token scope is modified, it overrides the existing one(s).

**editing a token**

```bash
curl -X PUT \
    -H "Authorization: Bearer <ADMIN token>" \
    "https://<your_host>/v0/tokens/<Token>?" \
    -d "name=test_new_name&description=this is a test token&scope=PIPES:READ:test_pipe&scope=DATASOURCES:CREATE"
```

### Request parameters

{% table %}
  * Key
  * Type
  * Description
  ---
  * token
  * String
  * Token. Ensure it has the `TOKENS` scope on it
  ---
  * name
  * String
  * Optional. Name of the token.
  ---
  * description
  * String
  * Optional. Markdown text with a description of the token.
  ---
  * scope
  * String
  * Optional. Scope(s) to set. Format is [SCOPE:TYPE[:arg][:filter]](#id2). New scope(s) will override existing ones.
{% /table %}

**Successful response**

```json
{
  "name": "test",
  "description": "this is a test token",
  "scopes": [
    { "type": "PIPES:READ", "resource": "test_pipe" },
    { "type": "DATASOURCES:CREATE" }
  ]
}
```
