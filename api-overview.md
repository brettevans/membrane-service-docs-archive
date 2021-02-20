---
id: api-overview
title: API Overview
sidebar_label: API Overview
---

This section contains the Membrane Service API Reference documentation. To make API requests, you will need to use capabilities as the authorization mechanism.

## Authentication and Authorization

Authentication and authorization with the Membrane Service is done through capabilities. A capability is an opaque, unique, and unguessable reference that grants the holder the authority to perform an action. In a capability-based system, authentication and authorization are never separate from each other. Therefore, when you make a request to the Membrane Service, you are at once authenticated and authorized by virtue of making a valid request.

Membrane Service capabilities are sent in the HTTP Authorization header using a Bearer token as described in [RFC6750](https://tools.ietf.org/html/rfc6750). For example:

```plaintext
Authorization: Bearer CPBLTY1-_PIPQQCNf1SSv6zHcEWjMdMlHtjlel1OvFBlZjOJ5Us-S2CBLDpAS4hyWKhUjEvDwLMe7uBDo4QVTPxkIZTiwQ
```

All requests must be made over [HTTPS](http://en.wikipedia.org/wiki/HTTP_Secure). Requests over HTTP will fail.

## Errors

The Membrane Service uses standard [HTTP status codes](http://en.wikipedia.org/wiki/List_of_HTTP_status_codes) to communicate information about successful calls or any errors. `2xx` codes indicate success, `4xx` codes indicate errors due to the contents or format of the request, and `5xx` codes indicate errors within the Membrane Service itself.

| Status Code | Meaning
| ----------- | ------- |
| `200 OK`    | Request worked and data was returned
| `201 Created` | Request worked and resource was created
| `202 Accepted` | Request worked and has been accepted for eventual processing
| `400 Bad Request` | Usually due to a missing parameter or unexpected request format
| `401 Unauthorized` | The request is unauthorized, usually due to missing or revoked capability
| `404 Not Found` | The resource does not exist
| `409 Conflict` | The provided resource already exists
| `500 Internal Server Error`<br/>`502 Bad Gateway`<br/>`503 Service Unavailable` | Something went wrong within the Membrane Service

### Example Errors

```json
{
    "statusCode": 400,
    "error": "Bad Request",
    "message": "Missing configuration"
}
```
```json
{
    "statusCode": 503,
    "error": "Service Unavailable",
    "message": "Please try again soon"
}
```
