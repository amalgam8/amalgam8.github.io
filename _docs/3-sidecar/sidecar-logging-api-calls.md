---
layout: page
title: Logging
permalink: /docs/sidecar/sidecar-logging-api-calls/
category: Sidecar
order: 6
---

All logs pertaining to external API calls made by the Nginx
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

If you are using a configuration file, the following two options have to be
added to the root of the YAML file:

```yaml
log: true
logstash_server: logstash:8092
```

**Note 1:** The logstash environment variable needs to be enclosed in single
quotes.

**Note 2:** You can omit the logstash server details if you override the
`filebeat.yml` file to log directly to Elasticsearch. The `filebeat.yml`
file can be found in `/etc/filebeat/filebeat.yml`.
