# Amalgam8 Images

The following Amalgam8 images can be pulled from Dockerhub.

## Control plane images

```
amalgam8/a8-controller:0.1
amalgam8/a8-registry:0.1
```

## Sidecar images

```
amalgam8/a8-sidecar:0.1        # reg + proxy sidecar + app supervisor (ubuntu base image)
amalgam8/a8-sidecar:0.1-alpine # reg + proxy sidecar (alpine base usually used in k8s pods or for standalone gateway)
```

## Sample images

### Standalone app images (usually used with sidecars in k8s pods)

```
amalgam8/a8-examples-helloworld
amalgam8/a8-examples-bookinfo-productpage:v1
amalgam8/a8-examples-bookinfo-details:v1
amalgam8/a8-examples-bookinfo-reviews:v1
amalgam8/a8-examples-bookinfo-reviews:v2
amalgam8/a8-examples-bookinfo-reviews:v3
amalgam8/a8-examples-bookinfo-ratings:v1
```

### App images with sidecar (app is managed by sidecar supervisor)

```
amalgam8/a8-examples-helloworld-sidecar
amalgam8/a8-examples-bookinfo-productpage-sidecar:v1
amalgam8/a8-examples-bookinfo-details-sidecar:v1
amalgam8/a8-examples-bookinfo-reviews-sidecar:v1
amalgam8/a8-examples-bookinfo-reviews-sidecar:v2
amalgam8/a8-examples-bookinfo-reviews-sidecar:v3
amalgam8/a8-examples-bookinfo-ratings-sidecar:v1
```