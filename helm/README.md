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
helm install --set dbReplicas=5 --name aerospike-release aerospike/aerospike-enterprise
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
- If using mounted volumes to apply the `feature-key-file`, you can use `aerospikeFeatureKeyFilePath` in `values.yaml` to specify the file path accessible within the container.
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

### Expose Aerospike Cluster

Aerospike Cluster can be exposed to client applications outside the K8s network by enabling `host networking`.
With host networking enabled, pods will be able to access the node's network. Pod IP will be same as node IP. 

> With host networking enabled, the deployment will be limited to one pod per node. If the new pod is scheduled onto the same node, it will fail due to no free ports.

Use `platform` and `hostNetworking` options to completely expose the Aerospike cluster pods to external client applications.

For example,

```
helm install --set dbReplicas=4 \
	         --name aerospike-release aerospike/aerospike-enterprise \
			 --set-file featureKeyFilePath=/secrets/aerospike/features.conf \
			 --set hostNetworking=true \
			 --set platform=gke
```

Client applications can connect to the Aerospike cluster using instance's external IP (if available). The external IP (if available) of the instance on which the pod is scheduled will be set as [`alternate-access-address`](https://www.aerospike.com/docs/reference/configuration/index.html#alternate-access-address) in its `aerospike.conf`.

```
asadm -h <ExternalIP> -p 3000 --services-alternate
```

### Configuration

| Parameter                          | Description                                                           | Default Value                |
| -----------------------------------|:--------------------------------------------------------------------: |:----------------------------:|
| `namespace`                        | Kubernetes Namespace                                                  |  `default`                   |
| `dbReplicas`                       | Number of Aerospike nodes or pods in the cluster                      |   `3`                        |
| `terminationGracePeriodSeconds`    | Wait time to forceful shutdown of a container                         |   `120`                      |
| `image.repository`                 | Aerospike Server Docker Image                                         | `aerospike/aerospike-server-enterprise` |
| `image.tag`                        | Aerospike Server Docker Image Tag                                     | `4.7.0.3`                    |
| `toolsImage.repository`            | Aerospike Tools Docker Image                                          | `aerospike/aerospike-tools`  |
| `toolsImage.tag`                   | Aerospike Tools Docker Image Tag                                      | `3.22.0`                     |
| `aerospikeNamespace`               | Aerospike Namespace name                                              | `test`                       |
| `aerospikeNamespaceMemoryGB`       | Aerospike Namespace Memory in GB                                      | `1`                          |
| `aerospikeReplicationFactor`       | Aerospike Namespace Replication Factor                                | `2`                          |
| `aerospikeDefaultTTL`              | Aerospike Namespace Record default TTL                                | `0` (Never Expire)           |
| `aerospikeFeatureKeyFilePath`      | Aerospike `feature-key-file` path accessible from within the container (if using mounted volumes to share feature key file)| `not defined` |
| `autoRolloutConfig`		   	     | Rollout ConfigMap/Secrets changes on 'helm upgrade'    			     | `false`					   	|
| `hostNetworking`		 			 | Enable `hostNetwork`. Allows Pods to access host network.			 	 | `false`					   	|
| `platform`		 				 | Set platform. Use with `hostNetworking` configuration to enable client applications outside the network to connect to Aerospike Cluster. Supported values - `eks` (AWS) or `gke` (GCP) or `none`    		 | `none`					   	|
| `antiAffinity`		 			 | Enable `PodAntiAffinity` rule to schedule one pod per node. Supported values - `off`, `soft`, `hard` | `off` |
| `antiAffinityWeight`		 		 | 'weight' in range 1-100 for "soft" antiAffinity option    			 | `1`					   		|
| `affinity`		 				 | Define custom `nodeAffinity`/`podAffinity`/`podAntiAffinity` rules	 | `{}` (nil)				   	|
| `aerospikeSecurity.enabled`		 	| To use Aerospike access control configuration    					 | `false`					   	|
| `aerospikeSecurity.aerospikeUsername` | Aerospike User Name to access the cluster     					 | `admin`					   	|
| `aerospikeSecurity.aerospikePassword`	| Aerospike User Password to access the cluster     				 | `admin`					   	|
| `persistenceStorage`               | Define Peristent Volumes to be used (Map - to define multiple volumes)| `{}` (nil)                   |
| `volumes`                          | Define volumes section and template to be used                        | `volume.mountPath: /opt/aerospike/data`,<br />`volume.name: datadir`,<br />`volume.template: emptyDir: {}`|
| `resources`                        | Resource configuration (`requests` and `limits`)                      | `{}` (nil)                   |
| `confFilePath`                     | Custom aerospike.conf file path on helm client machine (To be used during the runtime, `helm install` .. etc)| `not defined`|
| `featureKeyFilePath`               | Feature Key File (Enterprise License) file location on helm client machine (To be used during the runtime, `helm install` .. etc). | `not defined` |

Note that the namespace related configurations (`aerospikeNamespace`, `aerospikeNamespaceMemoryGB`, `aerospikeReplicationFactor` and `aerospikeDefaultTTL`) are intended for default single namespace configuration.

If using multiple namespaces, these config items can be ignored and a separate `aerospike.conf` file or template with multiple namespace configuration can be used.
