# HTTP Status Code Reference

## Success

| Code | Name | Use For |
|------|------|---------|
| 200 | OK | GET, PUT, PATCH (with response body) |
| 201 | Created | POST (include Location header) |
| 204 | No Content | DELETE, PUT (no response body) |

## Client Errors

| Code | Name | Use For |
|------|------|---------|
| 400 | Bad Request | Validation failure, malformed JSON |
| 401 | Unauthorized | Missing or invalid authentication |
| 403 | Forbidden | Authenticated but not authorized |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Duplicate entry, state conflict |
| 422 | Unprocessable Entity | Semantically invalid (valid JSON, bad data) |
| 429 | Too Many Requests | Rate limit exceeded |

## Server Errors

| Code | Name | Use For |
|------|------|---------|
| 500 | Internal Server Error | Unexpected failure (never expose details) |
| 502 | Bad Gateway | Upstream service failed |
| 503 | Service Unavailable | Temporary overload, include Retry-After |
