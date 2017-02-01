---
layout: page
title: Configuration
permalink: /docs/sidecar-configuration.html
redirect_from: /docs/sidecar/sidecar-configuration-options/
category: Sidecar
order: 3
---

The following instructions apply to both Docker-based and Kubernetes-based
installations. Configuration options can be set via environment variables,
command line flags, or YAML configuration files.  *Order of precedence is
command line flags first, then environment variables, configuration files,
and lastly default values.*

To run the sidecar with a configuration file, use the following command:

```bash
a8sidecar -config /path/to/a8sidecar.yaml
```

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
| A8_HEALTHCHECKS | --healthchecks | healthchecks (additional details below) | comma separated list of health check URIs |  | no |
| A8_REGISTER | --register | register | enable automatic service registration and heartbeat | false | See note above |
| A8_PROXY | --proxy | proxy | enable automatic service discovery and load balancing across services using NGINX | false | See note above |
| A8_SUPERVISE | --supervise | supervise | **valid upto sidecar versions 0.4.0 only.** Manage application process. If application dies, sidecar process is killed as well. All arguments provided after the flags will be considered as part of the application invocation | false | no |
| A8_REGISTRY_URL | --registry_url | registry.url | registry URL |  | yes if `-register` is enabled |
| A8_REGISTRY_TOKEN | --registry_token | registry.token | registry auth token | | yes if `-register` is enabled and an auth mode is set |
| A8_REGISTRY_POLL | --registry_poll | registry.poll | interval for polling Registry | 15s | no |
| A8_CONTROLLER_URL | --controller_url | controller.url | controller URL |  | yes if `-proxy` is enabled |
| A8_CONTROLLER_TOKEN | --controller_token | controller.token | Auth token for Controller instance |  | yes if `-proxy` is enabled and an auth mode is set |
| A8_CONTROLLER_POLL | --controller_poll | controller.poll | interval for polling Controller | 15s | no |
| A8_DISCOVERY_ADAPTER | --discovery_adapter | discovery_adapter | Service discovery type, one of `amalgam8`, `kubernetes` or `eureka` | `amalgam8` | no |
| A8_RULES_ADAPTER | --rules_adapter | rules_adapter | Rules controller type, one of `amalgam8` or `kubernetes` | `amalgam8` | no |
| A8_KUBERNETES_URL | --kubernetes_url | kubernetes.url | Kubernetes API master URL | | yes, if `-discovery_adapter` or `-rules_adapter` are set to `kubernetes` |
| A8_KUBERNETES_TOKEN | --kubernetes_token | kubernetes.token | Kubernetes API master access token | | yes, if `-discovery_adapter` or `-rules_adapter` are set to `kubernetes` |
| A8_KUBERNETES_NAMESPACE | -- kubernetes_namespace | kubernetes.namespace | Kubernetes namespace to use | `default` | no |
| A8_KUBERNETES_POD_NAME | -- kubernetes_pod_name | kubernetes.podname | Pod name to use for automtic service and tag generation | | no |
|  | --help, -h | show help | | |
|  | --version, -v | print the version | | |
{:.table .table-bordered .table-condensed .table-striped}

## Configuration Topics

* [Kubernetes configuration](/docs/kubernetes-integration-intro.html#sidecar-config)
* [Service registration](#service-registration)
* [Health checks](#health-checks)
* [Request routing](#request-routing)
* [Process supervision](#process-supervision)

### Service registration <a id="service-registration"></a>

Registration and heartbeat with the Amalgam8 service registry 
can be enabled by setting the following environment variables or 
setting the equivalent fields in the config file:

```bash
A8_SERVICE=service_name:service_version_tag,some_other_tag
A8_ENDPOINT_PORT=port_where_service_is_listening
A8_ENDPOINT_TYPE=https
A8_REGISTER=true
A8_REGISTRY_URL=http://a8registryURL
```

Equivalent YAML config:

```yaml
#a8sidecar.yaml
service:
  name: service_name
  tags: 
    - service_version_tag
    - some_other_tag
  
endpoint:
  port: port_where_service_is_listening
  type: https

register: true
registry:
  url:   http://registry:8080
```

The default endpoint type is `http`. So, the environment variable `A8_ENDPOINT_TYPE` and the YAML key `type` under `endpoint` section can be omitted, when registering an `http` endpoint.


### Health checks <a id="health-checks"></a>

In addition to automatic service registration,
the sidecar can periodically check the health of the application through TCP, HTTP or simply by
running a specified command. _When the application fails to respond or returns an error, 
the sidecar will immediately unregister the service instance from the 
Amalgam8 registry._

Health checks can configured using environment variable in the following manner:

```bash
A8_HEALTHCHECKS=http://localhost:8080/health1,tcp://localhost:9090
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
  - type: tcp
    value: http://localhost:9090
    interval: 30s
    timeout: 3s
  - type: file
    value: /opt/check_my_app.sh
    interval: 10s
    timeout: 5s
```

where
* `type` indicates the type of health check. **Sidecar versions upto 0.4.0 support HTTP health checks only. 0.4.1 and higher support TCP, and command based health checks as well.**
* `value` indicates the health check URL, or TCP endpoint or the path to the script to be executed. In case of HTTP, the sidecar will access the specified URL via the GET method only.
* `interval` indicates the frequency of the health check. Time intervals can be suffixed with `s`, `m` to indicate seconds or minutes.
* `timeout` indicates how long the sidecar will wait for the health check to execute.
* `code` indicates the HTTP response code that the sidecar should expect from the application in HTTP/HTTPS health checks and the exit code to expect from the command run in command health checks. 

**Note:** When using HTTPS endpoints, self-signed certificates will cause the health checks to fail as the connection attempt will fail in the first place.

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

### Process supervision <a id="process-supervision"></a>

When packaging the sidecar along with the application in the same docker container,
the sidecar can be made to act as the supervisor process that launches the application
process and other agents in the container. Upon graceful termination (or a crash) of the
application process, the sidecar will also terminate, effectively causing the container to
terminate as well.

**For sidecar versions 0.4.1 and above:** Sidecar can start and supervise any number 
of application processes. If one of the applications dies, supervisor can be configured 
to terminate the sidecar process as well. 

All arguments provided after the flags will be considered a part of a single application
invocation.  By default, the sidecar process will exit if this application terminates. The following
`ENTRYPOINT` can be placed in a Dockerfile to start the sidecar process as well as one python application:

```bash
ENTRYPOINT ["a8sidecar", "python", "productpage.py", "9080", "http://localhost:6379" ]
```

More advanced supervision modes can be configured by providing a yaml config.  For instance, if you
wanted to run the [filebeat](https://www.elastic.co/products/beats/filebeat) log shipping agent and
a python application, the following yaml could be provided:

```yaml
commands:
  - cmd: [ "filebeat", "-c", "/etc/filebeat.yml" ]
    env: [ "GODEBUG=netdns=go" ]
    on_exit: ignore
  - cmd: [ "python", "productpage.py", "9080", "http://localhost:6379" ]
    on_exit: terminate
```

where
* `cmd` indicates the commands you want to run.
* `env` indicates any additional environment variables the application may need to run with.
* `on_exit` can have the values `ignore` or `terminate` which indicates what the sidecar 
process should do if this application terminates.

_ignore vs terminate_: When the `on_exit` field for a particular process is set to `ignore`, the sidecar
will not restart the process when it exits (even if the process exits with an error). When the `on_exit`
field is set to `terminate`, **all processes managed by the sidecar will be terminated and the sidecar 
will shutdown/exit.** If the sidecar is the primary process in a Docker container, i.e.

```bash
ENTRYPOINT ["a8sidecar", "--config", "sidecar.yaml" ]
```

then failure of one of the managed processes whose `on_exit` is set to `terminate` will cause the
container itself to terminate (after shutting down other processes in the container).

It is advisable to use the `on_exit: ignore` for non-essential helper processes in the container, while
the `on_exit: terminate` should be used for the main applicaton processes.


**For sidecar versions 0.4.0 and below:** Prior to 0.4.1, only a single process can be supervised
by the sidecar. To supervise a single process, specify the --supervise flag
in the command line. For example,

```bash
ENTRYPOINT ["a8sidecar", "python", "productpage.py", "9080", "http://localhost:6379" ]
```

and the equivalent YAML configuration:

```yaml
##Setting up app supervision
supervise: true
app: [ "python", "productpage.py", "9080", "http://localhost:6379" ]
```

**Dealing with daemons and child processes**: When the parent process managed by the sidecar exits,
the sidecar will automatically kill all the child processes that the parent process had spawned. If an application launches background processes and then exits gracefully, it will cause the sidecar to terminate all child processes and exit as well.

For example, run nginx in foreground with `daemon off;` instead of the default background mode.

```yaml
commands:
    # Disable daemons and let nginx run in the foreground.
  - cmd: [ "nginx", "-g", "daemon off;" ]
    on_exit: terminate
```

As another example, in the following configuration, the sidecar will kill all processes eventhough this may not be what is intended.

```yaml
commands:
  - cmd: [ "/opt/launchapps.sh" ]
    on_exit: terminate
```

where `launchapps.sh` is a simple script that launches the application and all the processes as background processes and exits immediately.

```bash
#!/bin/bash
nginx ## daemonizes, and nginx launches worker processes
python myappserver.py &
```

Instead, the prescribed method for launching these processes should be via the sidecar config file

```yaml
commands:
  - cmd: [ "nginx", "-g", "daemon off;" ]
    on_exit: terminate
  - cmd: [ "python", "myappserver.py" ]
  - on_exit: terminate
```

### A complete configuration file

The following is an example config file for a sidecar that supervises a python application called `productpage.py`, starts up the filebeat log shipping agent, registers the application with the service registry, monitors its health and proxies outbound requests to other services. If the application terminates, the sidecar kills other processes and exits, thereby causing the container to terminate.

```yaml
#registration
register: true
registry:
  url:   http://registry:8080
  poll:  5s

endpoint:
  port: 9080

#request proxying
proxy: true
controller:
  url:   http://controller:8080
  poll:  30s

#Process management
commands:
    #start the app process. If app terminates, terminate container
  - cmd: [ "python", "productpage.py", "9080", "http://localhost:6379" ]
    on_exit: terminate
  - cmd: [ "filebeat", "-c", "/etc/filebeat.yml" ]
    env: [ "GODEBUG=netdns=go" ]
    on_exit: ignore

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
