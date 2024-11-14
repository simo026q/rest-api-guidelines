# REST Guidelines

> Note: These guidelines are heavily opinionated and may not suit all use cases. Adapt them to your requirements.

## Table of contents

1. [HTTP Methods](HTTP-METHODS.md)
2. [Pagination, filtering, and sorting](PAGINATION-FILTERING-AND-SORTING.md)
3. [Error handling](#error-handling--status-codes)
4. [Best practices & performance considerations](#best-practices--performance-considerations)

### Versioning strategy

| Version | Description                                                                 |
|---------|-----------------------------------------------------------------------------|
| Major   | Breaking changes or significant changes to the structure of the guidelines. |
| Minor   | New guidelines or significant changes to existing guidelines.               |
| Patch   | Minor changes and improvements to the language and formatting.              |

## Error handling & status codes

- **Validation errors:** If the client provides invalid parameters (e.g., `page` < 1 or unsupported filter fields), return a `400 Bad Request` with a detailed error message.
- **Range errors:** If `page` exceeds the `totalPages`, respond with a `404 Not Found` or a clear `400 Bad Request`.

## Best practices & performance considerations

- **Cache control:** Use appropriate cache headers to optimize API performance.
- **Limit maximum `pageSize`:** To prevent performance degradation, set a maximum value for `pageSize` (e.g., `100` items).
- **Cursor-based pagination:** Consider using cursor-based pagination for large datasets to improve performance.