---
layout: page
title: Deployment Options
permalink: /docs/sidecar-deployment-options.html
category: Sidecar
order: 1
---

There are two deployment options for the sidecar.

# As a Helper Process

This usage model applies to any container-based environment and requires
that the sidecar binaries be bundled along with the application in the same
Docker image.  Binary tarballs are available at the
[Amalgam8 releases](https://github.com/amalgam8/amalgam8/releases) page on
GitHub. To use the binary tarballs, add the following line to your
Dockerfile:

```dockerfile
RUN curl -sSL https://github.com/amalgam8/amalgam8/releases/download/${VERSION}/a8sidecar.sh | sh
```

If you are using `wget`

```dockerfile
RUN wget -qO- https://github.com/amalgam8/amalgam8/releases/download/${VERSION}/a8sidecar.sh | sh
```

Replace ${VERSION} with the specific version of Amalgam8 you would
like. The set of releases are available
[here](https://github.com/amalgam8/amalgam8/releases).

**Note:** The above script is intended for Debian and Ubuntu based docker images only.

Make sure to launch the sidecar along with your application. The sidecar
can be launched using the `a8sidecar` command. Refer to the
[configuration options](/docs/sidecar-configuration.html) page for details on how to
configure the sidecar.

A typical docker file will look like this:

```dockerfile
FROM <somebaseimage>
RUN curl -sSL https://github.com/amalgam8/amalgam8/releases/download/${VERSION}/a8sidecar.sh | sh

## Install your app stuff here

## script_to_launch_sidecar_and_app
```

The sidecar can also serve as a supervisor process that automatically
starts up your application. When the application dies, the sidecar exits
with the same exit code as the application, causing the container to
terminate as well. 

To use the sidecar to manage your application, set the `ENTRYPOINT` of your
`Dockerfile` as follows:

```dockerfile
ENTRYPOINT ["a8sidecar", "--config", "path/to/sidecar/config.yaml"]
```

In the sidecar's configuration file, provide the following arguments:

```yaml
supervise: true
app: [ "python", "helloworld.py" ]
```


# As a Helper Container in the Same Pod

This model applies to Kubernetes-based deployments. Each pod would have the
microservice running in one container and the sidecar in another.  No
changes are needed to the application's Dockerfile. Modify your service's
YAML file to launch the sidecar as another container in the same pod as
your application container. The latest version of the sidecar is available
in Docker Hub in two formats:

*  `amalgam8/a8-sidecar` - ubuntu-based version
*  `amalgam8/a8-sidecar:alpine` - alpine linux based version

A typical Kubernetes YAML file for a service would look like the following:

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: service_name
  labels:
    name: service_name
spec:
  replicas: 3
  selector:
    name: service_name
  template:
    metadata:
      labels:
        name: service_name
    spec:
      containers:
      - name: myMicroservice
        image: myMicroservice_docker_image
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: <SomePort>
      - name: sidecar
        image: amalgam8/a8-sidecar:alpine
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 6379
        env:
        - name: A8_REGISTER
          value: "true"
        - name: A8_PROXY
          value: "true"
        - name: A8_CONTROLLER_URL
          value: http://controllerURL
        - name: A8_CONTROLLER_POLL
          value: 5s
        - name: A8_REGISTRY_POLL
          value: 5s
        - name: A8_REGISTRY_URL
          value: http://registryURL
        - name: A8_SERVICE
          value: service_name:service_version,other_tags
        - name: A8_ENDPOINT_HOST
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        - name: A8_ENDPOINT_PORT
          value: "<Port_where_service_is_listening>"
```
