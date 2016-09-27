---
layout: page
title: Configuration
permalink: /docs/sidecar/sidecar-configuration-options/
category: Sidecar
order: 3
---

The following instructions apply to both Docker-based and Kubernetes-based
installations. Configuration options can be set via environment variables,
command line flags, or YAML configuration files.  *Order of precedence is
command line flags first, then environment variables, configuration files,
and lastly default values.*

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
| A8_REGISTER | --register | register | enable automatic service registration and heartbeat | false | See note above |
| A8_PROXY | --proxy | proxy | enable automatic service discovery and load balancing across services using NGINX | false | See note above |
| A8_LOG | --log | log | enable logging of outgoing requests through proxy using FileBeat | false | no |
| A8_SUPERVISE | --supervise | supervise | Manage application process. If application dies, sidecar process is killed as well. All arguments provided after the flags will be considered as part of the application invocation | false | no |
| A8_REGISTRY_URL | --registry_url | registry.url | registry URL |  | yes if `-register` is enabled |
| A8_REGISTRY_TOKEN | --registry_token | registry.token | registry auth token | | yes if `-register` is enabled and an auth mode is set |
| A8_REGISTRY_POLL | --registry_poll | registry.poll | interval for polling Registry | 15s | no |
| A8_CONTROLLER_URL | --controller_url | controller.url | controller URL |  | yes if `-proxy` is enabled |
| A8_CONTROLLER_TOKEN | --controller_token | controller.token | Auth token for Controller instance |  | yes if `-proxy` is enabled and an auth mode is set |
| A8_CONTROLLER_POLL | --controller_poll | controller.poll | interval for polling Controller | 15s | no |
| A8_LOGSTASH_SERVER | --logstash_server | logstash_server | logstash target for nginx logs |  | yes if `-log` is enabled |
|  | --help, -h | show help | | |
|  | --version, -v | print the version | | |
{:.table .table-bordered .table-condensed .table-striped}

## Example configuration file

```yaml
proxy: true

registry:
  url:   http://registry:8080
  poll:  5s
  
controller:
  url:   http://controller:8080
  poll:  30s
  
supervise: true
app: [ "python", "helloworld.py ]

log: true
logstash_server: logstash:8092
```

## Example Configuration using Environment Variables

**Automatic service registration:** Registration and heartbeat with the
Amalgam8 service registry can be enabled by setting the following
environment variables or setting the equivalent fields in the config file:

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

**Intelligent Request Routing:** For microservices that make outbound calls
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
