---
layout: page
title: Usage
permalink: /docs/sidecar/usage/
category: Sidecar
order: 2
---

The communication model between a microservice, its sidecar (irrespective
of the deployment model) and the target microservice is shown below:

![Communication between app and sidecar](/docs/figures/amalgam8-sidecar-communication-model.svg)

When you want to make API calls to other microservices from your
application, you should call the sidecar at localhost:6379.  The format of
the API call is [http://localhost:6379/serviceName/endpoint]()

where the `serviceName` is the service name that was used when launching
the target microservice (the `A8_SERVICE` environment variable), and the
endpoint is the API endpoint exposed by the target microservice.

For example, to invoke the `getItem` API in the `catalog` microservice,
your microservice would simply invoke the API via the URL:
[http://localhost:6379/catalog/getItem?id=123]().

Note that service versions are not part of the URL. The choice of the
service version (e.g., catalog:v1, catalog:v2, etc.), will be done
dynamically by the sidecar, based on routing rules set by the Amalgam8
controller.

**Note**: You can enable HTTPS between microservices by customizing the
Nginx config files in `/etc/nginx/amalgam8-services.conf`. For more
information on setting up HTTPS with Nginx, please refer to the official
[Nginx guide](https://www.nginx.com/resources/admin-guide/nginx-https-upstreams/)
