# Migrating from Version 1

There are a few breaking changes between version 1 and version 2.

## `AuthenticationError` Moved

`proxmoxer.backends.https.AuthenticationError` was moved to `proxmoxer.AuthenticationError` (the class itself is the same).
Any imports or references to `proxmoxer.backends.https.AuthenticationError` should be changed to `proxmoxer.AuthenticationError`.

## `ProxmoxResourceBase` Removed

While this should be a fully internal change, the `ProxmoxResourceBase` class was removed. Use `ProxmoxResource` instead.

## Removed `ProxmoxHTTPTicketAuth`

The `auth_token` and `csrf_token` arguments are no longer supported. If an existing (still valid) token needs to be used, you can pass the token as `password` and proxmoxer will attempt to renew the ticket and retrieve a new token and CSRF token.

## Testing changed to pytest

The unit testing framework was transitioned from nose to pytest. Any integration with the proxmoxer tests should now call pytest.
