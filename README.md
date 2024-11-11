# REST Guidelines

**NOTE:** These guidelines are heavily opinionated and may not be suitable for all use cases. Please adapt them to your specific requirements.

## Query parameters for pagination, filtering and sorting

### Pagination

- `page` - 1-based page number
- `pageSize` - number of items per page

Provide default values for `page` and `pageSize` if not specified by the client. The default values should be `page=1` and `pageSize=10` (may vary based on resource).

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

### JSON Response format

#### Structure

```json
{
  "data": [],
  "pagination": {
    "page": 2,
    "pageSize": 10,
    "totalItems": 100,
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
- `pagination`: Pagination metadata
  - `page`: Current page number
  - `pageSize`: Number of items per page
  - `totalItems`: Total number of items
  - `totalPages`: Total number of pages
- `links`: Links to navigate through the paginated results (optional)
  - `first`: Link to the first page
  - `prev`: Link to the previous page (if available)
  - `self`: Link to the current page
  - `next`: Link to the next page (if available)
  - `last`: Link to the last page

### Headers

#### Request headers

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