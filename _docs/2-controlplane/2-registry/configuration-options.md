---
layout: page
title: Configuration
permalink: /docs/control-plane-registry-configuration.html
redirect_from: /docs/control-plane/registry/registry-configuration-options/
category: Control Plane
subcategory: Service Registry
order: 1
---

The following instructions apply to both Docker-based and Kubernetes-based
installations. The Amalgam8 Registry supports a number of configuration
options, most of which are set through environment variables. The environment
variables can be set via command line flags as well.

The following environment variables are available. All of them are optional.

| Environment Variable | Flag Name                   | Description | Default Value |
|:---------------------|:----------------------------|:------------|:--------------|
| `A8_API_PORT` | `--api_port` | API port number | 8080 |
| `A8_LOG_LEVEL` | `--log_level` | Logging level. Supported values are: `debug`, `info`, `warn`, `error`, `fatal`, `panic` | `debug` |
| `A8_LOG_FORMAT` | `--log_format` | Logging format. Supported values are: `text`, `json`, `logstash` | `text` |
| `A8_NAMESPACE_CAPACITY` | `--namespace_capacity` | maximum number of instances that may be registered in a namespace | -1 (no capacity limit) |  
| `A8_DEFAULT_TTL` | `--default_ttl` | Registry default instance time-to-live (TTL) | 30s |
| `A8_MIN_TTL` | `--min_ttl` | Minimum TTL that may be specified during registration | 10s | 
| `A8_MAX_TTL` | `--max_ttl` | Maximum TTL that may be specified during registration | 10m |
| `A8_AUTH_MODE` | `--auth_mode` | Authentication modes. Supported values are: `trusted`, `jwt` | none (no isolation) |
| `A8_JWT_SECRET` | `--jwt_secret` | Secret key for JWT authentication | none (must be set if `A8_AUTH_MODE` is `jwt`) |
| `A8_REQUIRE_HTTPS` | `--require_https` | Require clients to use HTTPS for API calls | `false` |
{:.table .table-bordered .table-condensed .table-striped}

## Authentication and Multi-tenancy

The Amalgam8 Registry supports multi-tenancy by isolating each tenant into
a separate namespace.  Refer to the
[Authentication](/docs/control-plane-authentication.html) section 
for further details.

## Clustering

Amalgam8 Registry uses a memory only storage solution, without persistency
(although different storage backends can be implemented). To provide HA and
scale, the Registry can be run in a cluster and supports replication
between cluster members. To use a persistent storage backend, see the
section [Persistent Backend Storage](#persistent_storage).

Peer discovery currently uses a shared volume between all members. The
volume must be mounted RW into each container.  We are exploring
alternative discovery mechanisms.


| Environment Variable | Flag Name                   | Description | Default Value |
|:---------------------|:----------------------------|:------------|:--------------|
| `A8_CLUSTER_SIZE` | `--cluster_size` | Cluster minimal healthy size, peers detecting a lower value will log errors | 1 (standalone) |
| `A8_CLUSTER_DIR` | `--cluster_dir` | Filesystem directory for cluster membership | none, must be specified for clustering to work |
| `A8_REPLICATION` | `--replication` | Enable replication between cluster members | `false` |
| `A8_REPLICATION_PORT` | `--replication_port` | Replication port number | 6100 |
| `A8_SYNC_TIMEOUT` | `--sync_timeout` | Timeout for establishing connections to peers for replication | 30s |
{:.table .table-bordered .table-condensed .table-striped}

## <a name="persistent_storage"></a> Persistent Backend Storage using Redis

Amalgam8 Registry supports a Redis backend for storing instance information
as an alternative to the memory only clustering and replication option.

| Environment Variable | Flag Name                   | Description | Default Value |
|:---------------------|:----------------------------|:------------|:--------------|
| `A8_STORE` | `--store` | Backing store to use to persist Registry instance information. Supported values are: `redis`, `inmem`  | inmem (in memory) |
| `A8_STORE_ADDRESS` | `--store_address` | Address of the Redis server | none, must be specified to use a Redis store |
| `A8_STORE_PASSWORD` | `--store_password` | Password for the Redis backend | none, assumes no password set for the Redis server |
{:.table .table-bordered .table-condensed .table-striped}

## Catalog Extensions

The Amalgam8 Registry supports read-only catalogs extensions. The content
of each catalog extension (e.g., Kubernetes, Docker-Swarm, Eureka,
FileSystem, etc) is read by the Registry and returned to the user along
with the content of the Registry itself.

| Environment Variable | Flag Name                   | Description | Default Value |
|:---------------------|:----------------------------|:------------|:--------------|
| `A8_K8S_URL` | `--k8s_url` | Enable kubernetes catalog and specify the API server | (none) |
| `A8_K8S_TOKEN` | `--k8s_token` | Kubernetes API token | (none) |
| `A8_EUREKA_URL` | `--eureka_url` | Enable eureka catalog and specify the API server. Multiple API servers can be specified using multiple flags | (none) |
| `A8_FS_CATALOG` | `--fs_catalog` | Enable FileSystem catalog and specify the directory of the config files. The format of the file names in the directory should be `<namespace>.conf`. See [File Catalog](/docs/control-plane-registry-external-services.html) for more information | (none) |
{:.table .table-bordered .table-condensed .table-striped}
