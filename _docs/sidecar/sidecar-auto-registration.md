---
layout: page
title: Automatic Service Registration
permalink: /docs/sidecar/sidecar-auto-service-registration/
category: Sidecar
order: 4
---

Registration and heartbeat with the
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
