# temporary-contents

[temporary-contents](https://github.com/qlik-trial/temporary-contents) is the service responsible for managing the temporary-contents resource (and `/v1/temp-contents`).

## Introduction

This chart bootstraps a temporary-contents deployment on a [Kubernetes](http://kubernetes.io) cluster using the [Helm](https://helm.sh) package manager.

## Installing the Chart

To install the chart with the release name `my-release`:

```console
helm install --name my-release qlik/temporary-contents
```

The command deploys temporary-contents on the Kubernetes cluster in the default configuration. The [configuration](#configuration) section lists the parameters that can be configured during installation.

## Uninstalling the Chart

To uninstall/delete the `my-release` deployment:

```console
helm delete my-release
```

The command removes all the Kubernetes components associated with the chart and deletes the release.

## Configuration

The following tables lists the configurable parameters of the temporary-contents chart and their default values.

| Parameter                                     | Description                                                                                                                                   | Default                                                          |
| --------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| `auth.enabled`                                | Toggle JWT validation using retrieved keys from the configured JWKS endpoint.                                                                 | `true`                                                           |
| `auth.jwksURI`                                | The endpoint to retrieve the JWKS                                                                                                             | `http://%s-keys:8080/v1/keys/qlik.api.internal`                  |
| `auth.jwtAud`                                 | The expected `audience` value within the JWT claims                                                                                           | `qlik.api.internal`                                              |
| `auth.jwtIss`                                 | The expected `issuer` value within the JWT claims                                                                                             | `qlik.api.internal`                                              |
| `accessControl.enabled`                       | Toggle access control for download requests.                                                                                                  | `false`                                                          |
| `accessControl.url`                           | URL where the policy decision service is available.                                                                                           | `http://{{ .Release.Name }}-policy-decisions:5080`               |
| `encryption.enabled`                          | If `true` will use the encryption service to retrieve encryption keys, if `false` will use static key from a local loopback service           | `false`                                                          |
| `encryption.url`                              | URL where the encryption service is available at.                                                                                             | `http://{{ .Release.Name }}-encryption:8080`                     |
| `tokenAuth.enabled`                           | Toggle token authentication for Service-to-service JWT communication                                                                          | `false`                                                          |
| `tokenAuth.privateKey`                        | The private key for the self-signed service JWT                                                                                               | `...`                                                            |
| `tokenAuth.kid`                               | Unique identifier for the key (public)                                                                                                        | `ckZJbjOzS1zWHMV8XqX5Q_YKWx2A4FIuGM-Ac8PF4aA`                    |
| `tokenAuth.url`                               | Token validation endpoint URL                                                                                                                 | `"http://{{ .Release.Name }}-edge-auth:8080/v1/internal-tokens"` |
| `image.registry`                              | Image registry                                                                                                                                | `qliktech-docker.jfrog.io`                                       |
| `image.repository`                            | Image repository name (i.e. just the name without the registry)                                                                               | `temporary-contents`                                             |
| `image.tag`                                   | Image version                                                                                                                                 | `1.2.0`                                                          |
| `image.pullPolicy`                            | Image pull policy                                                                                                                             | `Always` if `image.tag` is `latest`, else `IfNotPresent`         |
| `imagePullSecrets`                            | A list of secret names for accessing private image registries                                                                                 | `[{name: "artifactory-docker-secret"}]`                          |
| `metrics.prometheus.enabled`                  | whether prometheus metrics are enabled                                                                                                        | `true`                                                           |
| `persistence.enabled`                         | Use persistent volume claim for persitence                                                                                                    | `true`                                                           |
| `persistence.accessMode`                      | Persistence access mode                                                                                                                       | `ReadWriteMany`                                                  |
| `persistence.storageClass`                    | Storage class of backing persistent volume claim                                                                                              | `nil`                                                            |
| `persistence.size`                            | Persistence volume size                                                                                                                       | `5Gi`                                                            |
| `persistence.existingClaim`                   | If defined, PersistentVolumeClaim is not created and the chart uses this claim                                                                | `nil`                                                            |
| `persistence.internalStorageClass.enabled`    | Use an internal StorageClass                                                                                                                  | `false`                                                          |
| `persistence.internalStorageClass.definition` | Definition of the internal StorageClass. Configuration includes provider and parameters. Only needed if the internal StorageClass is enabled. | `{}`                                                             |
| `replicaCount`                                | Number of temporary-contents replicas                                                                                                                     | `1`                                                              |
| `terminationGracePeriodSeconds`               | Number of seconds to wait during pod termination after sending SIGTERM until SIGKILL                                                          | `30`                                                             |
| `service.type`                                | Service type                                                                                                                                  | `ClusterIP`                                                      |
| `service.port`                                | Service listen port                                                                                                                           | `6080`                                                           |
| `ingress.class`                               | The `kubernetes.io/ingress.class` to use                                                                                                      | `nginx`                                                          |
| `ingress.authURL`                             | The URL to use for nginx's `auth-url` configuration to authenticate `/api` requests                                                           | `http://{.Release.Name}-edge-auth.{.Release.Namespace}.svc.cluster.local:8080/v1/auth` |
| `metrics.prometheus.enabled`                  | Whether prometheus metrics are enabled                                                                                                        | `true`                                                           |
| `mongodb.enabled`                             | Enable Mongodb as a chart dependency                                                                                                          | `true`                                                           |
| `mongodb.uri`                                 | If the mongodb chart dependency isn't used, specify the URI path to mongo                                                                     |                                                                  |
| `mongodb.uriSecretName`                       | Name of secret to mount for mongo URI. The secret must have the `mongodb-uri` key                                                             | `{release.Name}-mongoconfig`                                     |

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`.

Alternatively, a YAML file that specifies the values for the parameters can be provided while installing the chart. For example,

```console
helm install --name my-release -f values.yaml qlik/temporary-contents
```

> **Tip**: You can use the default [values.yaml](values.yaml)

## Persistence

Temporary-contents stores files in the /qlik/temp-contents directory in the container.
The chart mounts a Persistent Volume Claim at this location. The volume is created using dynamic volume provisioning. In order to disable this functionality you can change the values.yaml to disable persistence and use an emptyDir instead.

### Access mode

The access mode can be changed depending on the deployed usage scenario and the underlying storage:

- Default is `ReadWriteOnce`

### StorageClass

Dynamic provisioning is configured by

1. Create a StorageClass
2. Install the helm chart

```console
$ helm install --name my-release --set persitence.storageClass=STORAGE_CLASS_NAME temporary-contents
```

#### Using the internal StorageClass

Normally a StorgeClass is created by an administrator prior to installing the temporary contents helm chart. An option to create the StorageClass as part of the helm chart is provided to simplify certain deployment scenarios.
Example configuration:

```yaml
persistence:
  enabled: true
  storageClass: "qlik-docs"
  internalStorageClass:
    enabled: true
    definition:
      provisioner: kubernetes.io/no-provisioner
      parameters: {}
      reclaimPolicy: Retain
      mountOptions: {}
```

### Existing PersistentVolumeClaims

You can also configure and external PersistentVolumeClaim by setting the claim name in the existingClaim parameter.

1. Create a Persistent Volume
2. Create a Persistent Volume Claim
3. Install chart.

```console
$ helm install --name my-release --set persitence.existingClaim=PVC_QLIKAPPS temporary-contents
```
