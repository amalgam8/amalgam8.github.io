---
layout: page
title: Configuration
permalink: /docs/controller/controller-configuration-options/
category: Controller
order: 1
---

The following instructions apply to both Docker-based and Kubernetes-based
installations. The Amalgam8 Controller supports a number of configuration
options, most of which are set through environment variables. The environment
variables can be set via command line flags as well.

The following environment variables are available. All of them are optional.

| Environment Key | Flag Name                   | Description | Default Value |
|:----------------|:----------------------------|:------------|:--------------|
| A8_API_PORT | --api_port | API port | 8080 |
| A8_ENCRYPTION_KEY | --encryption_key | secret key | abcdefghijklmnop |
| A8_DATABASE_TYPE |  --database_type |	database type | memory |
| A8_DATABASE_USERNAME | --database_username | database username | |
| A8_DATABASE_PASSWORD | --database_password | database password | |
| A8_DATABASE_HOST | --database_host | database host | |
| A8_LOG_LEVEL | --log_level | logging level (debug, info, warn, error, fatal, panic) | info |
| A8_AUTH_MODE | --auth_mode | Authentication modes. Supported values are: 'trusted', 'jwt'" | |
| A8_JWT_SECRET | --jwt_secret | Secret key for JWT authentication | |
| A8_REQUIRE_HTTPS | --require_https | Require clients to use HTTPS for API calls | |
| | --help, -h | show help | |
| | --version, -v | print the version | |
{:.table .table-bordered .table-condensed .table-striped}

## Example Configuration using Environment Variables

```bash
A8_LOG_LEVEL=debug
```

## Persistent Storage Backend

Controller by default will use in-memory storage.  To persist created rules
to a redis storage backend, enable the following options:

```bash
A8_DATABASE_TYPE=redis
A8_DATABASE_HOST=redis://redis:6379
```

The `redis:alpine` image from dockerhub does not have a password set by default
(redis does not use a username).  If the redis instance in use is configured
with a password, set the options above and this additional option as well:

```bash
A8_DATABASE_PASSWORD=mypassword
```
