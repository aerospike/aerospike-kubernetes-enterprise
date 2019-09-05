# Helm chart for Aerospike (EE) on Kubernetes

Implements a dynamically scalable Aerospike cluster using Kubernetes StatefulSets.


## Pre Requisites

- Kubernetes 1.8+

## Usage:

### Add Aerospike repository

```sh
helm repo add aerospike https://aerospike.github.io/aerospike-kubernetes-enterprise
```

### Install the chart

```sh
helm install --name aerospike-release aerospike/aerospike-enterprise
```

You can also set the configuration values as defined in `values.yaml` using `--set` option or provide a `values.yaml` file during `helm install`.

For example,

```sh
helm install --set dBReplicas=5 --name aerospike-release aerospike/aerospike-enterprise
```

### Apply your own aerospike.conf file or template

- To override the default `aerospike.template.conf`, set `confFilePath` to point to your own custom `aerospike.conf` file or template.
- Note that `confFilePath` should be a path on your machine where `helm` client is running.
- The custom `aerospike.conf` file or template must contain `# mesh-seed-placeholder` in `heartbeat` configuration to populate mesh configuration during peer discovery. For example,

```
....
	heartbeat {

        address any
		mode mesh
		port 3002

		# mesh-seed-placeholder

		interval 150
		timeout 10
	}
.....
```

- Use `confFilePath` during `helm install` with `--set-file` option.
```
helm install --name aerospike-release --set-file confFilePath=/tmp/aerospike_templates/aerospike.template.conf aerospike/aerospike-enterprise
```

### Apply feature-key-file (Enterprise licence) file

- To supply `feature-key-file` during the deployment, use `featureKeyFilePath` to point to your `features.conf` licence file during `helm install`.
- Note that `featureKeyFilePath` should be a path on your machine where `helm` client is running.
- If using mounted volumes to apply the `feature-key-file`, you can use `aerospikeFeatureKeyFile` in `values.yaml` to specify the file path accessible within the container. 
- If using this repo directly, you can also add the license file to `files/` directory which will be added to the ConfigMap automatically with `helm install`.

Example,

```
helm install --name aerospike-release --set-file featureKeyFilePath=/secrets/aerospike/features.conf aerospike/aerospike-enterprise
```

### Storage configuration

You can configure multiple volume mounts (filesystem type) or device mounts (raw block device) or both in `values.yaml`. Please check below [configuration section](#configuration) and `values.yaml` file in this repo for more details.


### Test Output:

```sh
NAME:   as-release
LAST DEPLOYED: Thu Sep  5 22:05:02 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/ConfigMap
NAME             DATA  AGE
as-release-conf  4     3m2s

==> v1/Pod(related)
NAME                               READY  STATUS    RESTARTS  AGE
as-release-aerospike-enterprise-0  1/1    Running   0         3m2s
as-release-aerospike-enterprise-1  0/1    Init:0/1  0         90s

==> v1/Service
NAME                             TYPE       CLUSTER-IP  EXTERNAL-IP  PORT(S)   AGE
as-release-aerospike-enterprise  ClusterIP  None        <none>       3000/TCP  3m2s

==> v1/StatefulSet
NAME                             READY  AGE
as-release-aerospike-enterprise  1/5    3m2s
```

```sh
$ helm list
NAME      	REVISION	UPDATED                 	STATUS  	CHART                     	APP VERSION	NAMESPACE
as-release	1       	Thu Sep  5 22:05:02 2019	DEPLOYED	aerospike-enterprise-4.6.0	4.6.0.2    	default  
```

### Configuration

| Parameter                          | Description                                                           | Default Value                |
| -----------------------------------|:--------------------------------------------------------------------: |:----------------------------:|
| `namespace`                        | Kubernetes Namespace                                                  |  `default`                   |
| `dBReplicas`                       | Number of Aerospike nodes or pods in the cluster                      |   `1`                        |
| `terminationGracePeriodSeconds`    | Wait time to forceful shutdown of a container                         |    `30`                      |
| `image.repository`                 | Aerospike Server Docker Image                                         | `aerospike/aerospike-server-enterprise` |
| `image.tag`                        | Aerospike Server Docker Image Tag                                     | `4.6.0.2`                    |
| `toolsImage.repository`            | Aerospike Tools Docker Image                                          | `aerospike/aerospike-tools`  |
| `toolsImage.tag`                   | Aerospike Tools Docker Image Tag                                      | `3.21.1`                     |
| `aerospikeNamespace`               | Aerospike Namespace name                                              | `test`                       |
| `aerospikeNamespaceMemoryGB`       | Aerospike Namespace Memory in GB                                      | `1`                          |
| `aerospikeReplicationFactor`       | Aerospike Namespace Replication Factor                                | `2`                          |
| `aerospikeDefaultTTL`              | Aerospike Namespace Record default TTL                                | `30d` (days)                  |
| `persistenceStorage`               | Define Peristent Volumes to be used (Map - to define multiple volumes)| `{}` (nil)                   |
| `volumes`                          | Define volumes section and template to be used                        | `volume.mountPath: /opt/aerospike/data`,<br />`volume.name: datadir`,<br />`volume.template: emptyDir: {}`|
| `resources`                        | Resource configuration (`requests` and `limits`)                      | `{}` (nil)                   |
| `confFilePath`                     | Custom aerospike.conf file path on helm client machine (To be used during the runtime, `helm install` .. etc)| `not defined`|
| `featureKeyFilePath`               | Feature Key File (Enterprise License) file location on helm client machine (To be used during the runtime, `helm install` .. etc). | `not defined` |
