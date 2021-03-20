# Specification

## Authentication

Authentication on the 42 API works with [OAuth2](https://oauth.net/2/).

OAuth2 is a protocol that lets external apps request authorization to private details in a user’s 42 account without getting their password. This is preferred over a basic authentication because tokens can be limited to specific types of data, and can be revoked by users at any time.

All developers need to [register their application](https://profile.intra.42.fr/oauth/applications/new) before getting started. A registered OAuth application is assigned a unique **Client ID** and **Client Secret**. The Client Secret should not be shared.

Once the access token aquired, it can be passed in the URL with the `access_token` parameter, or in the `Authorization: Bearer <TOKEN>` header field.

## Errors

The 42 API uses the following error codes:

Error Code | Meaning
---------- | -------
400 | The request is malformed
401 | Unauthorized
403 | Forbidden
404 | Page or resource not found
422 | Unprocessable entity
500 | We have a problem with our server. Please try again later.
Connection refused | Most likely cause is not using `HTTPS`.

## Scopes

Applications can have different access scopes.

Authorization scopes are a way to determine to what extent the client can use resources located in the provider.

When the client requests the authorization it specifies in which scope they would like to be authorized. This information is then displayed to the user — resource owner — and they can decide whether or not they accept the given application to be able to act in specified scopes.

> Requesting a resource with wrong or insufficient scopes will return a `403 Forbidden` response, with more details in the `WWW-Authenticate` response header. For example, for an application without the `projects` scope:

```http
POST https://api.intra.42.fr/v2/topics/4242/messages

HTTP/1.1 403 Forbidden
Cache-Control: no-store
Content-Type: application/json; charset=utf-8
Pragma: no-cache
Transfer-Encoding: chunked
Vary: Origin
WWW-Authenticate: Bearer realm="42 API", error="insufficient scope", error_description="The action need the following scopes: [forum]"
X-Application-Id: 7
X-Application-Name: Brobot
X-Application-Roles: None
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
X-Meta-Request-Version: 0.4.0
X-Rack-CORS: preflight-hit; no-origin
X-Request-Id: bb64ca44-142b-46a6-b590-af95e2e05e66
X-Runtime: 0.045248
X-XSS-Protection: 1; mode=block
```

```json
{
  "error": "Forbidden",
  "message": "Insufficient scope. The action need the following scopes: [forum] (Create, update and destroy topics and messages)"
}
```

## Pagination

The 42 API paginates all resources on the index method.

Requests that return multiple items will be paginated to 30 items by default. You have two ways to specify further pages:

- The `page` parameter. You can also set a custom page size (up to 100) with the `per_page` parameter.
- The `page[number]` paramater with the `page[size]` parameter.

Note that for technical reasons not all endpoints can go up to 100 on the the `per_page` / `page[size]` parameter.

> The `Link` HTTP response header contains pagination data with `first`, `previous`, `next` and `last` raw pages links when available, under the format:

```http
link: <http://api.intra.42.fr/v2/{Resource}?page=X+1>; rel="next", <http://api.intra.42.fr/v2/{Resource}?page=X-1>; rel="prev", <http://api.intra.42.fr/v2/{Resource}?page=1>; rel="first", <http://api.intra.42.fr/v2/{Resource}?page=X+n>; rel="last"
```

There are also:

- `X-Page`: a header field, which contains the current page.
- `X-Per-Page`: a header field, which contains the current pagination length.
- `X-Total`: a header field, which contains the count of pages.

## Filtering

The `filter` query parameter can be used to filter a collection on one or several fields for one or several values. The `filter` parameter takes the field to filter as a key, and the values to filter as the value. Multiples values must be comma-separated (`,`).

For example, the following is a request for all users who have their piscine in 2013, but only in September or July:

```http
GET /users?filter[pool_year]=2013&filter[pool_month]=september,july HTTP/1.1
```

## Sorting

All index endpoints support multiple sort fields by allowing comma-separated (`,`) sort fields, they are applied in the order specified.

The sort order for each sort field is ascending unless it is prefixed with a minus (U+002D HYPHEN-MINUS, `-`), in which case it is descending.

> For example,
```http
GET /users?sort=kind,-login
```
> will return the users sorted by kind. Any users with the same kind will then be sorted by their login in descending alphabetical order.

## Rate limiting

By default, your applications is limited to **2 requests per second** and **1200 requests per hour**.

## JSON-API format

The API actually support (in an alpha-stage) the [JSON-API](http://jsonapi.org/) format specification. You can request a JSON-API by specifying the request `ContentType` as `application/vnd.api+json`.

## Getting information about your token

```shell
curl -H "Authorization: Bearer <YOUR_ACCESS_TOKEN>" https://api.intra.42.fr/oauth/token/info
```

```json
{
  "resource_owner_id": 74,
  "scopes": ["public"],
  "expires_in_seconds": 7174,
  "application":
  {
    "uid": "3089cd94d72cc9109800a5eea5218ed4c3e891ec1784874944225878b95867f9"
  },
  "created_at":1439460680
}
```

If you want to know more about your token, you can fetch `https://api.intra.42.fr/oauth/token/info`.
