---
layout: page
title: Making API Calls using the Sidecar
permalink: /docs/sidecar/using-the-amalgam8-sidecar/
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

## Runtime Configurations for the Sidecar <a id="runtime"></a>

The following instructions apply to both Docker-based and Kubernetes-based
installations. Configuration options can be set via environment variables,
command line flags, or YAML configuration files.  *Order of precedence is
command line flags first, then environmenmt variables, configuration files,
and lastly default values.*

The following table lists all the configuration options, their equivalent
environment variable name, the command line switch and the field in the
YAML file. 

| Environment Variable | Flag Name                   | YAML Key | Description | Default Value |Required|
|:---------------------|:----------------------------|:---------|:------------|:--------------|--------|
| A8_CONFIG | --config | | Path to a file to load configuration from | | no |
| A8_LOG_LEVEL | --log_level | log_level | Logging level (debug, info, warn, error, fatal, panic) | info | no |
| A8_SERVICE | --service | service.name & service.tags | service name to register with, optionally followed by a colon and a comma-separated list of tags | | yes |
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

### Example configuration file

```yaml
register: true
proxy: true

service:
  name: helloworld
  tags: 
    - v1
    - somethingelse
  
endpoint:
  host: 172.10.10.1
  port: 9080
  type: https

registry:
  url:   http://registry:8080
  token: abcdef
  poll:  10s
  
controller:
  url:   http://controller:8080
  token: abcdef
  poll:  30s
  
supervise: true
app: [ "python", "helloworld.py ]

log: true
logstash_server: logstash:8092

log_level: debug
```

### Configurations Options

**Automatic service registration:** Registration and heartbeat with the
Amalgam8 service registry can be enabled by setting the following
environment variables or setting the equivalent fields in the config file:

```bash
A8_REGISTER=true
A8_REGISTRY_URL=http://a8registryURL
A8_SERVICE=service_name:service_version_tag
A8_ENDPOINT_PORT=port_where_service_is_listening
A8_ENDPOINT_TYPE=http|https|tcp|udp|user
```

**Intelligent Request Routing:** For microservices that make outbound calls
to other microservices, service discovery and client-side load balancing,
version and content-based routing can be enabled by the following options:

```bash
A8_PROXY=true
A8_CONTROLLER_URL=http://a8controllerURL
```

**Logging:** All logs pertaining to external API calls made by the Nginx
proxy will be stored in `/var/log/nginx/a8_access.log` and
`/var/log/nginx/error.log`. The access logs are stored in JSON format. Note
that there is **no support for log rotation**. If you have a monitoring and
logging system in place, it is advisable to propagate the request logs to
your log storage system in order to take advantage of Amalgam8 features
like resilience testing.

The sidecar installation comes preconfigured with
[Filebeat](https://www.elastic.co/products/beats/filebeat) that can be
configured automatically to ship the Nginx access logs to a Logstash
server, which in turn propagates the logs to elasticsearch. If you wish to
use the filebeat system for log processing, make sure to have Elasticsearch
and Logstash services available in your application deployment. The
following two environment variables enable the filebeat process:

```bash
A8_LOG=true
A8_LOGSTASH_SERVER='logstash_server:port'
```

**Note 1:** The logstash environment variable needs to be enclosed in single
quotes.

**Note 2:** You can omit the logstash server details if you override the
`filebeat.yml` file to log directly to Elasticsearch. The `filebeat.yml`
file can be found in `/etc/filebeat/filebeat.yml`.
