---
layout: page
title: Logging
permalink: /docs/sidecar-logging-api-calls.html
category: Sidecar
order: 6
---

All logs pertaining to external API calls made by the Nginx
proxy are be stored in `/var/log/nginx/a8_access.log` and
`/var/log/nginx/error.log`. The access logs are stored in JSON format.

Note that there is **no support for log rotation**. If you have a
monitoring and logging system in place, it is advisable to propagate the
request logs to your log storage system in order to take advantage of
Amalgam8 features like resilience testing.
