# REST Guidelines

> Note: These guidelines are heavily opinionated and may not suit all use cases. Adapt them to your requirements.

## Table of contents

### Versioning strategy

| Version | Description                                                                 |
|---------|-----------------------------------------------------------------------------|
| Major   | Breaking changes or significant changes to the structure of the guidelines. |
| Minor   | New guidelines or significant changes to existing guidelines.               |
| Patch   | Minor changes and improvements to the language and formatting.              |

## HTTP Methods

### Global response codes

| Status Code                 | Description                                        | Response                                                                                                                   |
|-----------------------------|----------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------|
| `400 Bad Request`           | Malformed syntax (e.g., invalid query parameters). | [RFC 7807](https://datatracker.ietf.org/doc/html/rfc7807) problem detail object with a `errors` extension.                 |
| `401 Unauthorized`          | Missing or invalid `Authorization` header.         | [RFC 7807](https://datatracker.ietf.org/doc/html/rfc7807) problem detail object or empty body.                             |
| `500 Internal Server Error` | Internal server error.                             | [RFC 7807](https://datatracker.ietf.org/doc/html/rfc7807) problem detail object with `traceId` and `requestId` extensions. |

### GET

- **Purpose:** Retrieve resource(s).
- **Idempotent:** Yes.
- **Cacheable:** Yes.

#### Response codes

| Status Code      | Description                                                  | Response                                                                                       |
|------------------|--------------------------------------------------------------|------------------------------------------------------------------------------------------------|
| `200 OK`         | Resource(s) found.                                           | Returns the resource or a collection of resources.                                             |
| `204 No Content` | Empty collection.                                            | Successful response for when the requested collection of resources is empty.                   |
| `404 Not Found`  | Resource not found.                                          | [RFC 7807](https://datatracker.ietf.org/doc/html/rfc7807) problem detail object or empty body. |
| `403 Forbidden`  | Access denied (use `404` to prevent leaking sensitive info). | [RFC 7807](https://datatracker.ietf.org/doc/html/rfc7807) problem detail object or empty body. |

### POST

- **Purpose:** Create a resource.
- **Idempotent:** No.
- **Cacheable:** No.

#### Response codes

| Status Code                  | Description                                                                   | Response                                                                                                                                                                                                             |
|------------------------------|-------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `201 Created`                | Resource created successfully.                                                | Returns the resource when the `Prefer` header is `return=representation`. Always include a [`Location` header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Location) with the URI of the new resource. |
| `200 OK` or `204 No Content` | Resource created successfully but the resource cannot be identified by a URI. | Use the [`Prefer` header](#prefer-header) to determine the response format.                                                                                                                                          |
| `403 Forbidden`              | Authenticated user does not have permission to create the resource.           | [RFC 7807](https://datatracker.ietf.org/doc/html/rfc7807) problem detail object or empty body.                                                                                                                       |

### PUT

- **Purpose:** Update (or optionally create) a resource.
- **Idempotent:** Yes.
- **Cacheable:** No.

#### Response codes

| Status Code                  | Description                                                  | Response                                                                                                                                   |
|------------------------------|--------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
| `200 OK` or `204 No Content` | Resource updated successfully.                               | Use the [`Prefer` header](#prefer-header) to determine the response format.                                                                |
| `201 Created`                | New resource created.                                        | Always include a [`Location` header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Location) with the URI of the new resource. |
| `404 Not Found`              | Resource not found.                                          | [RFC 7807](https://datatracker.ietf.org/doc/html/rfc7807) problem detail object or empty body.                                             |
| `403 Forbidden`              | Access denied (use `404` to prevent leaking sensitive info). | [RFC 7807](https://datatracker.ietf.org/doc/html/rfc7807) problem detail object or empty body.                                             |

### DELETE

- **Purpose:** Delete a resource.
- **Idempotent:** Yes.
- **Cacheable:** No.

> Consideration: Use soft delete (marking the resource as deleted) instead of hard delete (removing the resource from the database).

#### Response codes

| Status Code                  | Description                                                                              | Response                                                                                       |
|------------------------------|------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------|
| `200 OK` or `204 No Content` | Resource deleted successfully.                                                           | Use the [`Prefer` header](#prefer-header) to determine the response format.                    |
| `202 Accepted`               | The request has been accepted for processing, but the processing has not been completed. | Empty body.                                                                                    |
| `404 Not Found`              | Resource not found or user does not have access.                                         | [RFC 7807](https://datatracker.ietf.org/doc/html/rfc7807) problem detail object or empty body. |
| `403 Forbidden`              | Access denied (use `404` to prevent leaking sensitive info).                             | [RFC 7807](https://datatracker.ietf.org/doc/html/rfc7807) problem detail object or empty body. |

### PATCH

- **Purpose:** Partial update to a resource.
- **Idempotent:** Yes.
- **Cacheable:** No.

#### Request body example

```json
[
  { "op": "test",  "path": "/a/b/c",  "value": "foo"  }, 
  { "op": "remove",  "path": "/a/b/c"  }, 
  { "op": "add",  "path": "/a/b/c",  "value": [ "foo", "bar" ] }, 
  { "op": "replace", "path": "/a/b/c",  "value": 42 }, 
  { "op": "move",  "from": "/a/b/c",  "path": "/a/b/d" }, 
  { "op": "copy", "from": "/a/b/d",  "path": "/a/b/e" }
]
```

#### Response codes

| Status Code                  | Description                                                  | Response                                                                                       |
|------------------------------|--------------------------------------------------------------|------------------------------------------------------------------------------------------------|
| `200 OK` or `204 No Content` | Resource updated successfully.                               | Use the [`Prefer` header](#prefer-header) to determine the response format.                    |
| `404 Not Found`              | Resource not found or user does not have access.             | [RFC 7807](https://datatracker.ietf.org/doc/html/rfc7807) problem detail object or empty body. |
| `403 Forbidden`              | Access denied (use `404` to prevent leaking sensitive info). | [RFC 7807](https://datatracker.ietf.org/doc/html/rfc7807) problem detail object or empty body. |

## Query parameters for pagination, filtering and sorting

### Pagination

| Parameter  | Description               | Default Value   |
|------------|---------------------------|-----------------|
| `page`     | Page number (1-based).    | `1`             |
| `pageSize` | Number of items per page. | `10` suggested. |

### Filtering

#### Simple filtering

Simple filtering is usable for most filtering requirements.

- **Flexible parameters:** Use query parameters that represent filterable fields.
  - Example of simple filtering: `GET /resources?status=active&category=books`
  - Example of range-based filtering: `GET /resources?minPrice=10&maxPrice=100`
  - Example of exclusion filtering: `GET /resources?excludeStatus=inactive`
- **Multiple values:** Use comma-separated values for filtering on multiple values of the same field.
  - Example: `GET /resources?category=books,electronics`
- **Date filtering:** Use ISO 8601 date format for date filtering.
  - Example: `GET /resources?createdAfter=2023-01-01T00:00:00Z&createdBefore=2023-12-31T23:59:59Z`
- **Boolean filtering:** Use `true` or `false` for boolean filtering.
  - Example: `GET /resources?active=true`
- **Filtering on nested fields:** Use dot notation for filtering on nested fields.
  - Example: `GET /resources?author.name=John`

#### Complex filtering

For complex filtering requirements, consider using a `filter` parameter with a query language like [OData](https://www.odata.org/).

- **Syntax:** Use a `filter` parameter with a query language expression.
  - Example: `GET /resources?filter=status eq 'active' and price lt 100`

Read more on [Microsoft's API guidelines](https://github.com/microsoft/api-guidelines/blob/vNext/graph/articles/collections.md#7-filtering).

### Sorting

- **Syntax:** Use a `sort` parameter, with fields and optional direction
  - Example: `GET /resources?sort=-price,name` (sort by price descending and name ascending)

The default behavior should depend on the resource and should be documented.

### JSON pagination response format

#### Structure

```json
{
  "data": [],
  "metadata": {
    "page": 2,
    "pageSize": 10,
    "totalCount": 100,
    "totalPages": 10
  },
  "links": {
    "first": "/resources?page=1&pageSize=10",
    "prev": "/resources?page=1&pageSize=10",
    "self": "/resources?page=2&pageSize=10",
    "next": "/resources?page=3&pageSize=10",
    "last": "/resources?page=10&pageSize=10"
  }
}
```

#### Fields

- `data`: Array of resources
- `metadata`: Pagination metadata
  - `page`: Current page number
  - `pageSize`: Number of items per page
  - `totalCount`: Total number of items
  - `totalPages`: Total number of pages
- `links`: Links to navigate through the paginated results (optional)
  - `first`: Link to the first page
  - `prev`: Link to the previous page (if available)
  - `self`: Link to the current page
  - `next`: Link to the next page (if available)
  - `last`: Link to the last page

#### Example: C# model

```cs
public sealed class PaginationResponse<T>(IEnumerable<T> data, PaginationMetadata metadata, PaginationLinks? links = null)
{
    [JsonPropertyName("data")]
    public IEnumerable<T> Data { get; } = data;

    [JsonPropertyName("metadata")]
    public PaginationMetadata Metadata { get; } = metadata;

    [JsonPropertyName("links")]
    public PaginationLinks? Links { get; } = links;
}

public sealed class PaginationMetadata(int page, int pageSize, int totalCount)
{
    [JsonPropertyName("page")]
    public int Page { get; } = page;

    [JsonPropertyName("pageSize")]
    public int PageSize { get; } = pageSize;

    [JsonPropertyName("totalCount")]
    public int TotalCount { get; } = totalCount;

    [JsonPropertyName("totalPages")]
    public int TotalPages => (int)Math.Ceiling((double)TotalCount / PageSize);
}

public sealed class PaginationLinks(string first, string? previous, string self, string? next, string last)
{
    [JsonPropertyName("first")]
    public string First { get; } = first;

    [JsonPropertyName("prev")]
    public string? Previous { get; } = previous;

    [JsonPropertyName("self")]
    public string Self { get; } = self;

    [JsonPropertyName("next")]
    public string? Next { get; } = next;

    [JsonPropertyName("last")]
    public string Last { get; } = last;
}
```

### Headers

#### Request headers

##### Prefer header

- `Prefer` header: Use the `Prefer` header to request the server to return a minimal representation of the resource. This is not applicable for `GET` requests.
  - Example: `Prefer: return=representation`
  - Example: `Prefer: return=minimal`

#### Response headers

##### Pagination

- `X-Total-Count`: Total number of items
- `Link`: A standard header for pagination links, following the format specified in [RFC 8288](https://tools.ietf.org/html/rfc8288).

### Error handling & status codes

- **Validation errors:** If the client provides invalid parameters (e.g., `page` < 1 or unsupported filter fields), return a `400 Bad Request` with a detailed error message.
- **Range errors:** If `page` exceeds the `totalPages`, respond with a `404 Not Found` or a clear `400 Bad Request`.

### Best practices & performance considerations

- **Cache control:** Use appropriate cache headers to optimize API performance.
- **Limit maximum `pageSize`:** To prevent performance degradation, set a maximum value for `pageSize` (e.g., `100` items).
- **Cursor-based pagination:** Consider using cursor-based pagination for large datasets to improve performance.