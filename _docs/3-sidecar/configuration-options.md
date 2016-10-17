---
layout: page
title: Configuration
permalink: /docs/sidecar-configuration.html
category: Sidecar
order: 3
---

The following instructions apply to both Docker-based and Kubernetes-based
installations. Configuration options can be set via environment variables,
command line flags, or YAML configuration files.  *Order of precedence is
command line flags first, then environment variables, configuration files,
and lastly default values.*

## Options Reference

The following table lists all the configuration options, their equivalent
environment variable name, the command line switch and the field in the
YAML file. 

| Environment Variable | Flag Name                   | YAML Key | Description | Default Value |Required|
|:---------------------|:----------------------------|:---------|:------------|:--------------|--------|
| A8_CONFIG | --config | | Path to a file to load configuration from | | no |
| A8_LOG_LEVEL | --log_level | log_level | Logging level (debug, info, warn, error, fatal, panic) | info | no |
| A8_SERVICE | --service | service.name & service.tags | name of the service provided by this application, optionally followed by a colon and a comma-separated list of tags | | yes |
| A8_ENDPOINT_HOST | --endpoint_host | endpoint.host | service endpoint IP or hostname. Defaults to the IP (e.g., container) where the sidecar is running | optional |
| A8_ENDPOINT_PORT | --endpoint_port | endpoint.port | service endpoint port |  | yes |
| A8_ENDPOINT_TYPE | --endpoint_type | endpoint.type | service endpoint type (http, https, udp, tcp, user) | http | no |
| A8_HEALTHCHECKS | --healthchecks | healthchecks (additional details below) | comma separated list of health check URIs (http only) |  | no |
| A8_REGISTER | --register | register | enable automatic service registration and heartbeat | false | See note above |
| A8_PROXY | --proxy | proxy | enable automatic service discovery and load balancing across services using NGINX | false | See note above |
| A8_SUPERVISE | --supervise | supervise | Manage application process. If application dies, sidecar process is killed as well. All arguments provided after the flags will be considered as part of the application invocation | false | no |
| A8_REGISTRY_URL | --registry_url | registry.url | registry URL |  | yes if `-register` is enabled |
| A8_REGISTRY_TOKEN | --registry_token | registry.token | registry auth token | | yes if `-register` is enabled and an auth mode is set |
| A8_REGISTRY_POLL | --registry_poll | registry.poll | interval for polling Registry | 15s | no |
| A8_CONTROLLER_URL | --controller_url | controller.url | controller URL |  | yes if `-proxy` is enabled |
| A8_CONTROLLER_TOKEN | --controller_token | controller.token | Auth token for Controller instance |  | yes if `-proxy` is enabled and an auth mode is set |
| A8_CONTROLLER_POLL | --controller_poll | controller.poll | interval for polling Controller | 15s | no |
|  | --help, -h | show help | | |
|  | --version, -v | print the version | | |
{:.table .table-bordered .table-condensed .table-striped}

## Example configurations

* [Service registration](#service-registration)
* [Health checks](#health-checks)
* [Request routing](#request-routing)

### Service registration <a id="service-registration"></a>

Registration and heartbeat with the Amalgam8 service registry 
can be enabled by setting the following environment variables or 
setting the equivalent fields in the config file:

```bash
A8_SERVICE=service_name:service_version_tag
A8_ENDPOINT_PORT=port_where_service_is_listening
A8_ENDPOINT_TYPE=http|https|tcp|udp|user
A8_REGISTER=true
A8_REGISTRY_URL=http://a8registryURL
```

Equivalent YAML config:

```yaml
service:
  name: service_name
  tags: 
    - service_version_tag
    - some_other_tag
  
endpoint:
  port: port_where_service_is_listening
  type: (http|https|tcp|udp|user)

register: true
registry:
  url:   http://registry:8080
```

  
### Health checks <a id="health-checks"></a>

In addition to automatic service registration,
the sidecar can periodically check the health of the application at specified 
HTTP endpoints. _When the application fails to respond or returns an error, 
the sidecar will immediately unregister the service instance from the 
Amalgam8 registry._

Health checks can configured using environment variable in the following manner:

```bash
A8_HEALTHCHECKS=http://localhost:8080/health1,http://localhost:9090/health2
```

Alternatively, a more expressive and customizable health check can be configured
in the YAML configuration file:

```yaml
healthchecks:
  - type: http
    value: http://localhost:8080/health1
    interval: 15s
    timeout: 5s
    code: 200
  - type: http
    value: http://localhost:9090/health2
    interval: 30s
    timeout: 3s
    code: 201
```

where
* `type` indicates the type of health check. Currently only http health checks are supported.
* `value` indicates the health check URL.
* `interval` indicates the frequency of the health check. Time intervals can be suffixed with `s`, `m` to indicate seconds or minutes.
* `timeout` indicates how long the sidecar will wait for the application to respond to the health check API call.
* `code` indicates the HTTP response code that the sidecar should expect from the application.

Note that the sidecar will access the specified URL via the GET method only.


### Request routing <a id="request-routing"></a>

For microservices that make outbound calls
to other microservices, service discovery and client-side load balancing,
version and content-based routing can be enabled by the following options:

```bash
A8_SERVICE=service_name:service_version_tag
A8_PROXY=true
A8_REGISTRY_URL=http://a8registryURL
A8_REGISTRY_POLL=5s
A8_CONTROLLER_URL=http://a8controllerURL
A8_CONTROLLER_POLL=5s
```

Equivalent YAML config:

```yaml
service:
  name: service_name
  tags: 
    - service_version_tag
    - some_other_tag

proxy: true

registry:
  url:   http://registry:8080
  poll:  5s
  
controller:
  url:   http://controller:8080
  poll:  5s
```

If you want to enable both request routing and registration, add the
following to the above config:

```bash
A8_REGISTER=true
```

or in YAML:

```yaml
register: true
```

### A complete configuration file

The following is an example config file for a sidecar that supervises a python application called `helloworld.py`, monitors its health and registers it with the service registry, while proxying requests to other services.

```yaml
registry:
  url:   http://registry:8080
  poll:  5s

proxy: true
controller:
  url:   http://controller:8080
  poll:  30s
 
##Setting up app supervision
supervise: true
app: [ "python", "helloworld.py ]

healthchecks:
  - type: http
    value: http://localhost:8080/health1
    interval: 15s
    timeout: 5s
    code: 200
  - type: http
    value: http://localhost:9090/health2
    interval: 30s
    timeout: 3s
    code: 201
```
