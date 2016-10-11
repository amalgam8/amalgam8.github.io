---
layout: page
title: File Catalog
permalink: /docs/control-plane/registry/file-catalog/
category: Control Plane
subcategory: Service Registry
order: 3
---

The File-based catalog extension allows the registry to pull static service
information from a file, in addition to supporting standard registration
via REST API.  This catalog is useful when you have services that do not
run the Amalgam8 protocol (e.g., a database service), and allows:

* discovery of these services using the standard Amalgam8 discovery REST
  API.
* specifying routing rules (such as fault injection for resiliency testing)
  on external services

To enable this catalog type you have to provide the `--fs_catalog` command
line flag and set it to the directory of the catalog configuration files
(one file per namespace).  The name of the configuration file should be
`<namespace>.json`, and it contains a single JSON object holding an array
of instances.  Each instance is formatted according to the registration
body defined in the
[API Swagger Documentation](/api/registry).  Service
instances registered using this method are persistent and do not need to
periodically heartbeat the registry in order to stay valid.

_Note: The file must be available to every instance of the registry
container._

## Configuration File Example

The below shows a sample configuration file adding two services: the first
(`db`) with a single endpoint, and the second (`ext-service`) having two
endpoints.

```json
{
    "instances": [
        {
            "service_name": "db",
            "endpoint": {
                "type" : "http",
                "value": "www.xdb-service.url"
             },
             "tags"  : [ "v1", "us-central" ]
        },
        {
            "service_name": "ext-service",
            "endpoint": {
                "type" : "tcp",
                "value": "192.168.1.2:8080"
             }
        },
        {
            "service_name": "ext-service",
            "endpoint": {
                "type" : "tcp",
                "value": "192.168.1.2:8443"
             },
             "tags"  : [ "secure" ]
        }        
    ]
}
```

## Example Usage

The following code snippets show how to register and access an external
database using Amalgam8.

### 1. Registring an External Service

The instance endpoint information is added in file called `default.json`
(the file name determines the registry's namespace used, in this case the
external database instance is registered in the default namespace).

The metadata fields can be used to specify arbitrary information such as access credentials.
Note that the Amalgam8 sidecar that proxying requests to external services does not read the 
metadata information.

```json
{
   "instances": [
       {
          "service_name": "cloudant",
          "endpoint" : {
             "type" : "https",
             "value" : "71a0c41e-6ea4-4abc-813a-5a08ad8bke3f.cloudant.com:443"
          },
          "tags" : [ "dal05" ],
          "metadata" : {
             "credentials": {
                "username": "71a0c41e-6ea4-4abc-813a-5a08ad8bke3f",
                "password": "5b0d3f2db7401917f9ba3714ba07245f213abba3ecb96632796487f0e553435c"
             }
          }
       }
    ]
}
```

The file is stored in a location accessible to the Amalgam8 registry, to
allow the registry to integrate the data on startup.  For example, the
following docker command will mount the host directory `/amalgam8/fs_catalog/` 
inside the container and load the catalogs in that directory (note
that we specify the path to the _directory_ containing the file(s), not the
full file path).

```bash
docker run -v /amalgam8/fs_catalog:/amalgam8/fs_catalog -p 8080:8080 amalgam8/a8-registry:latest --fs_catalog=/amalgam8/fs_catalog/
```

Once the registry is running, we can confirm the external service is
registered successfully, and a call to

```bash
curl localhost:8080/api/v1/instances
```

results in:

```json
{
  "instances": [
    {
      "id": "5aff5a8eefc853d1",
      "service_name": "cloudant",
      "endpoint": {
        "type": "https",
        "value": "71a0c41e-6ea4-4abc-813a-5a08ad8bke3f.cloudant.com:443"
      },
      "status": "UP",
      "metadata": {
        "credentials": {
           "username": "71a0c41e-6ea4-4abc-813a-5a08ad8bke3f",
           "password": "5b0d3f2db7401917f9ba3714ba07245f213abba3ecb96632796487f0e553435c"
        }
      },
      "last_heartbeat": "0001-01-01T00:00:00Z",
      "tags": [
        "dal05",
        "filesystem"
      ]
    }
  ]
}
```

### 2. Accessing the External Service

Once the external service instance is registered in the Amalgam8 registry,
you may specify routing rules to apply on the communication path.  However,
in order for the rules to be applied, communication must flow through the
sidecar's proxy. The client application must call the local service URL
instead of the original URL.

The recomended way (e.g., [Twelve-Factor Apps](https://12factor.net/)) is
to externalize all service URL's as configuration, allowing different
service endpoints to be used in development, testing and production.  Thus,
the original URL
`https://71a0c41e-6ea4-4abc-813a-5a08ad8bke3f.cloudant.com:443` should be
replaced in the application configuration with
`http://localhost:6379/cloudant`, matching the service name specified for
the instance.  The application should still provide all required headers
(e.g., authorization) when contacting the sidecar. These will be passed to
the upstream service unmodified.

```bash
curl -H "Authorization: basic NWIwZDNmMmRiNzQwMTkxN2Y5YmEzNzE0YmEwNzI0NWYyMTNlYmVmM2VjYjk2NjMyNzk2NDg3ZjBlNTUzNDM1Ywo=" http://localhost:6379/cloudant/<database>/<document_id>/
```
