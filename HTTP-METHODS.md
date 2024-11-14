# HTTP Methods

## Global response codes

| Status Code                 | Description                                        | Response                                                                                                                   |
|-----------------------------|----------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------|
| `400 Bad Request`           | Malformed syntax (e.g., invalid query parameters). | [RFC 7807](https://datatracker.ietf.org/doc/html/rfc7807) problem detail object with a `errors` extension.                 |
| `401 Unauthorized`          | Missing or invalid `Authorization` header.         | [RFC 7807](https://datatracker.ietf.org/doc/html/rfc7807) problem detail object or empty body.                             |
| `500 Internal Server Error` | Internal server error.                             | [RFC 7807](https://datatracker.ietf.org/doc/html/rfc7807) problem detail object with `traceId` and `requestId` extensions. |

## GET

- **Purpose:** Retrieve resource(s).
- **Idempotent:** Yes.
- **Cacheable:** Yes.

### Response codes

| Status Code      | Description                                                  | Response                                                                                       |
|------------------|--------------------------------------------------------------|------------------------------------------------------------------------------------------------|
| `200 OK`         | Resource(s) found.                                           | Returns the resource or a collection of resources.                                             |
| `204 No Content` | Empty collection.                                            | Successful response for when the requested collection of resources is empty.                   |
| `404 Not Found`  | Resource not found.                                          | [RFC 7807](https://datatracker.ietf.org/doc/html/rfc7807) problem detail object or empty body. |
| `403 Forbidden`  | Access denied (use `404` to prevent leaking sensitive info). | [RFC 7807](https://datatracker.ietf.org/doc/html/rfc7807) problem detail object or empty body. |

## POST

- **Purpose:** Create a resource.
- **Idempotent:** No.
- **Cacheable:** No.

### Response codes

| Status Code                  | Description                                                                   | Response                                                                                                                                                                                                             |
|------------------------------|-------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `201 Created`                | Resource created successfully.                                                | Returns the resource when the `Prefer` header is `return=representation`. Always include a [`Location` header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Location) with the URI of the new resource. |
| `200 OK` or `204 No Content` | Resource created successfully but the resource cannot be identified by a URI. | Use the [`Prefer` header](PAGINATION-FILTERING-AND-SORTING.md#prefer-header) to determine the response format.                                                                                                                                          |
| `403 Forbidden`              | Authenticated user does not have permission to create the resource.           | [RFC 7807](https://datatracker.ietf.org/doc/html/rfc7807) problem detail object or empty body.                                                                                                                       |

## PUT

- **Purpose:** Update (or optionally create) a resource.
- **Idempotent:** Yes.
- **Cacheable:** No.

### Response codes

| Status Code                  | Description                                                  | Response                                                                                                                                   |
|------------------------------|--------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
| `200 OK` or `204 No Content` | Resource updated successfully.                               | Use the [`Prefer` header](PAGINATION-FILTERING-AND-SORTING.md#prefer-header) to determine the response format.                                                |
| `201 Created`                | New resource created.                                        | Always include a [`Location` header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Location) with the URI of the new resource. |
| `404 Not Found`              | Resource not found.                                          | [RFC 7807](https://datatracker.ietf.org/doc/html/rfc7807) problem detail object or empty body.                                             |
| `403 Forbidden`              | Access denied (use `404` to prevent leaking sensitive info). | [RFC 7807](https://datatracker.ietf.org/doc/html/rfc7807) problem detail object or empty body.                                             |

## DELETE

- **Purpose:** Delete a resource.
- **Idempotent:** Yes.
- **Cacheable:** No.

> Consideration: Use soft delete (marking the resource as deleted) instead of hard delete (removing the resource from the database).

### Response codes

| Status Code                  | Description                                                                              | Response                                                                                       |
|------------------------------|------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------|
| `200 OK` or `204 No Content` | Resource deleted successfully.                                                           | Use the [`Prefer` header](PAGINATION-FILTERING-AND-SORTING.md#prefer-header) to determine the response format.                    |
| `202 Accepted`               | The request has been accepted for processing, but the processing has not been completed. | Empty body.                                                                                    |
| `404 Not Found`              | Resource not found or user does not have access.                                         | [RFC 7807](https://datatracker.ietf.org/doc/html/rfc7807) problem detail object or empty body. |
| `403 Forbidden`              | Access denied (use `404` to prevent leaking sensitive info).                             | [RFC 7807](https://datatracker.ietf.org/doc/html/rfc7807) problem detail object or empty body. |

## PATCH

- **Purpose:** Partial update to a resource.
- **Idempotent:** Yes.
- **Cacheable:** No.

### Request body example

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

### Response codes

| Status Code                  | Description                                                  | Response                                                                                       |
|------------------------------|--------------------------------------------------------------|------------------------------------------------------------------------------------------------|
| `200 OK` or `204 No Content` | Resource updated successfully.                               | Use the [`Prefer` header](PAGINATION-FILTERING-AND-SORTING.md#prefer-header) to determine the response format.                    |
| `404 Not Found`              | Resource not found or user does not have access.             | [RFC 7807](https://datatracker.ietf.org/doc/html/rfc7807) problem detail object or empty body. |
| `403 Forbidden`              | Access denied (use `404` to prevent leaking sensitive info). | [RFC 7807](https://datatracker.ietf.org/doc/html/rfc7807) problem detail object or empty body. |
