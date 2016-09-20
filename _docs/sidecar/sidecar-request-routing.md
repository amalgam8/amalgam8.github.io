---
layout: page
title: Intelligent Request Routing
permalink: /docs/sidecar/sidecar-intelligent-request-routing/
category: Sidecar
order: 5
---

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
