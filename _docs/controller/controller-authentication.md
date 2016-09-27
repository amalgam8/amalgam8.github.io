---
layout: page
title: Authentication Options & Multi-Tenancy
permalink: /docs/controller/controller-authentication/
category: Controller
order: 2
---

By default, the Controller operates in a single-tenant mode without
any authentication. In multi-tenant mode, it supports two authentication
mechanisms: a trusted auth mode for local testing and development
purposes, and a JWT auth mode for production deployments.

# Single-Tenant Mode

In single-tenant mode, there is no authentication header required in the
API calls to controller.  Any rules created in this mode are applied to
all sidecar instances polling this Amalgam8 controller instance.

**Configuration Options:** This mode runs by default and requires no
additional environment variables or command line flags be set at run time.

# Multi-Tenant Mode

In `trusted` and `jwt`, an Authentication header is required for all
API calls to the controller.  The header must be in the form
`Authentication: Bearer <TOKEN_VALUE>`.  Any rules created will be mapped
to the particular tenant associated with the provided Authentication header.
The following namespace authorization methods are supported and controlled
via the `A8_AUTH_MODE` environment variable (or `--auth_mode` flag):

## Trusted Authentication

Trusted authentication is intended for use in local testing and development
purposes.  The tenant namespace is retrieved directly from the Authorization
header. This provides namespace separation in a trusted environment (e.g.,
single tenant with multiple applications or environments).  Therefore
`TOKEN_VALUE` can be arbitrarily set by the user to any value.

**Configuration Options:** Enabled by the following environment variable:

```bash
A8_AUTH_MODE=trusted
```

## JWT Authentication

JWT authentication is intended for production environments. The `TOKEN_VALUE`
is the tenant's namespace value encoded in a signed JWT token claim.

**Configuration Options:** Enabled by the following environment variables
(both must be specified):

```bash
A8_AUTH_MODE=jwt
A8_JWT_SECRET=secretkey
```

### JWT Token Generation

JWT tokens can be generated and validated at [jwt.io](https://jwt.io).

**Header:**

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

**Payload:**

```json
{
  "exp": 1501184124,
  "namespace": "mynamespace"
}
```

The `Payload` can contain additional properties in the JSON but it must
contain the two above minimum.  Choose an `"exp"` time appropriate for the
desired level of security.  After this date/time, the token will no longer be
valid.

**Verify Signature:** Insert the `A8_JWT_SECRET` into the field here.

### Admin JWT Token

When JWT authentication is enabled for multi-tenancy, an administrator token can
be used to access information for any tenant namespace using the following steps:

1. Generate a token using the process above with `"namespace": "admin"`
2. Send the generated token from Step 1 in the authentication header: `Authentication: Bearer <admin_token>`
3. Send the ID for the desired tenant namespace in an additional header: `A8-Namespace: <namespace>`

