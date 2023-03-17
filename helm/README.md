# Helm chart for Aerospike (EE) on Kubernetes

## DEPRECATED
This Helm chart is no longer actively developed, instead use (https://artifacthub.io/packages/helm/aerospike/aerospike-kubernetes-operator)




## Pre Requisites

- Kubernetes 1.13+

## Usage:

### Add Aerospike repository

```sh
helm repo add aerospike https://aerospike.github.io/aerospike-kubernetes-enterprise
```

### Install the chart

```sh
helm install aerospike-release aerospike/aerospike-enterprise --set-file featureKeyFile=/secrets/aerospike/features.conf
```

All the configurations defined in [`values.yaml`](values.yaml) (or in the [configuration section](#configuration)) can be set using `--set` or `--set-file` option. A custom `values.yaml` file can also be provided using `-f` option.

For example,

```sh
helm install aerospike-release aerospike/aerospike-enterprise \
			 --set dbReplicas=5 \
			 --set-file featureKeyFile=/secrets/aerospike/features.conf
```

For Helm v2,

```sh
helm install --name aerospike-release aerospike/aerospike-enterprise \
			 --set dbReplicas=5 \
			 --set-file featureKeyFile=/secrets/aerospike/features.conf
```

### Apply custom Aerospike configuration

- To override the default `aerospike.template.conf`, set `aerospikeConfFile` to point to the custom `aerospike.conf` file or template.

	> `aerospikeConfFile` should be a file path on helm "client" machine (where the user is running the command `helm install`).

- `aerospikeConfFile` can be set using `--set-file` option,
	```sh
	helm install aerospike-release aerospike/aerospike-enterprise \
				 --set-file aerospikeConfFile=/tmp/aerospike_templates/aerospike.template.conf \
				 --set-file featureKeyFile=/secrets/aerospike/features.conf
	```

- Aerospike configuration file can also be passed in base64 encoded form. Use `aerospikeConfFileBase64` configuration to specify base64 encoded string of the Aerospike configuration file.
	```sh
	helm install aerospike-release aerospike/aerospike-enterprise \
				 --set aerospikeConfFileBase64=$(base64 /tmp/aerospike_templates/aerospike.template.conf) \
				 --set-file featureKeyFile=/secrets/aerospike/features.conf
	```

### Apply Aerospike enterprise licence

- To add a `feature-key-file` at the time of the deployment, set `featureKeyFile` to point to a `features.conf` licence file.

	> `featureKeyFile` should be a file path on helm "client" machine (where the user is running the command `helm install`).

	Example,
	```sh
	helm install aerospike-release aerospike/aerospike-enterprise \
				 --set-file featureKeyFile=/secrets/aerospike/features.conf
	```

- If a mounted volume is used to provide the `feature-key-file`, set `aerospikeFeatureKeyFilePath` to the file path accessible within the container.

- Aerospike license feature key file can also be passed in base64 encoded form. Use `featureKeyFileBase64` configuration to specify base64 encoded string of the Aerospike feature key file.
	```sh
	helm install aerospike-release aerospike/aerospike-enterprise \
				 --set featureKeyFileBase64=$(base64 /secrets/aerospike/features.conf)
	```
### Storage configuration

Aerospike helm chart allows multiple volume mounts (filesystem type) and device mounts (raw block device) to be configured and used with Aerospike Statefulset. Check below [configuration section](#configuration) or [`values.yaml`](https://github.com/aerospike/aerospike-kubernetes-enterprise/blob/master/helm/values.yaml) file for more details.

### Dynamic storage provisioning

Use `persistenceStorage` configuration to define the volumes that can be provisioned using a storageclass.

```yaml
persistenceStorage:
 - mountPath: /opt/aerospike/smd
   enabled: true
   name: smd-dir
   storageClass: standard
   accessMode: ReadWriteOnce
   volumeMode: Filesystem
   size: 1Gi
 - devicePath: /dev/sdb
   enabled: true
   name: data-dev
   storageClass: ssd
   accessMode: ReadWriteOnce
   size: 1Gi
   volumeMode: Block
```

### Static provisioning

Use `volumes` configuration to define volumes that needs to be mounted to aerospike pod. Use this option for accessing `secrets`, `hostPath` volumes, sharing `emptyDir` volumes etc.

```yaml
volumes:
 - mountPath: /opt/aerospike/data
   name: data-dir
   template:
     hostPath:
      path: /data
      type: Directory
 - mountPath: /secrets/
   name: secret-dir
   template:
     secret:
      secretName: aerospike-certs
```

### Expose Aerospike cluster

Aerospike Cluster can be exposed to client applications or to XDR clients which are outside the K8s network by enabling,
- Host Networking
- NodePort Services Per Pod
- LoadBalancer Services Per Pod
- ClusterIP Services with External IPs

### Host Networking

With host networking enabled, pods will be able to access the node's network. Pod IP will be same as node IP.

> With host networking enabled, the deployment will be limited to one pod per node. If the new pod is scheduled onto the same node, it will fail due to no free ports.

Use `hostNetwork.enabled` option to enable host networking and expose the Aerospike cluster pods to external client applications. Set `hostNetwork.useExternalIP` to allow applications to use external IP of the instances to connect to the aerospike cluster. The external IP of the instance will be added as [`alternate-access-address`](https://www.aerospike.com/docs/reference/configuration/index.html#alternate-access-address) in `aerospike.conf`.

For example,

```sh
helm install aerospike-release aerospike/aerospike-enterprise \
			 --set dbReplicas=4 \
			 --set-file featureKeyFile=/secrets/aerospike/features.conf \
			 --set hostNetwork.enabled=true \
			 --set hostNetwork.useExternalIP=true
```

For Helm v2,

```sh
helm install --name aerospike-release aerospike/aerospike-enterprise \
			 --set dbReplicas=4 \
			 --set-file featureKeyFile=/secrets/aerospike/features.conf \
			 --set hostNetwork.enabled=true \
			 --set hostNetwork.useExternalIP=true
```

Client applications can connect to the Aerospike cluster using instance's external IP (if available) or by using host IP.

```sh
asadm -h <ExternalIP> -p 3000 --services-alternate
```

### NodePort Services Per Pod

NodePort type exposes the Service on each K8s Node’s IP at a static port (the `NodePort`). Applications will be able to connect to the NodePort Service from outside the K8s cluster by using `<NodeIP>:<NodePort>`.

> With NodePort type services, it allows multiple aerospike pods per K8s host and at the same time expose each aerospike pod.

To enable `NodePort` services per pod, set `nodePortServices.enabled` option to `true`.
Aerospike helm chart will automatically create a `NodePort` type service for each aerospike pod at the time of deployment. Applications can connect to the Aerospike cluster using any one of the `<NodeIP>:<NodePort>` as a seed IP and Port.

Set `nodePortServices.useExternalIP` to allow applications to use external IP of the instances to connect to the aerospike cluster. The external IP of the instance will be added as [`alternate-access-address`](https://www.aerospike.com/docs/reference/configuration/index.html#alternate-access-address) in `aerospike.conf`.

> Use `helm upgrade` command to perform scale-up/scale-down when NodePort services are enabled.

Example,

```sh
helm install aerospike-release aerospike/aerospike-enterprise \
			 --set-file featureKeyFile=/secrets/aerospike/features.conf \
			 --set dbReplicas=5 \
			 --set nodePortServices.enabled=true \
			 --set nodePortServices.useExternalIP=true
```

For Helm v2,

```sh
helm install --name aerospike-release aerospike/aerospike-enterprise \
			 --set-file featureKeyFile=/secrets/aerospike/features.conf \
			 --set dbReplicas=5 \
			 --set nodePortServices.enabled=true \
			 --set nodePortServices.useExternalIP=true
```

### LoadBalancer Services Per Pod

LoadBalancer type exposes the service externally using the cloud provider’s load balancer. A new external network load balancer is provisioned.

> With LoadBalancer type services, it allows multiple aerospike pods per K8s host and at the same time expose each aerospike pod.

To enable `LoadBalancer` services per pod, set `loadBalancerServices.enabled` option to `true`.
Aerospike helm chart will automatically create a `LoadBalancer` type service for each aerospike pod at the time of deployment.

Applications can connect to the Aerospike cluster using `<LoadBalancerIngressIP>:<LoadBalancerPort>`.

```sh
asadm -h <LoadBalancerIngressIP> -p <LoadBalancerPort> --services-alternate
```

> Use `helm upgrade` command to perform scale-up/scale-down when LoadBalancer type services are enabled.

Example,

```sh
helm install aerospike-release aerospike/aerospike-enterprise \
			 --set-file featureKeyFile=/secrets/aerospike/features.conf \
			 --set dbReplicas=5 \
			 --set loadBalancerServices.enabled=true
```

For Helm v2,

```sh
helm install --name aerospike-release aerospike/aerospike-enterprise \
			 --set-file featureKeyFile=/secrets/aerospike/features.conf \
			 --set dbReplicas=5 \
			 --set loadBalancerServices.enabled=true
```

### ClusterIP Services with External IPs

`ClusterIP` type exposes the service on a cluster-internal IP. With external IPs set, the service can be accessed from an external endpoint.

With `externalIPServices.enabled` option set to `true`, Aerospike helm chart will create a `ClusterIP` type service for each aerospike pod at the time of deployment. The external endpoints can be specified using `externalIPServices.externalIPEndpoints` option. Each external endpoint can contain an `IP`, `Port` and an optional `TLSPort` (when TLS is enabled).

> Use `helm upgrade` command to perform scale-up/scale-down when ClusterIP-ExternalIP services are enabled.

> Number of endpoints should be equal to or more than the number of Aerospike pods (`dbReplicas`). Only one external IP and Port per Service can be specified.

Example,

```sh
helm install aerospike-release aerospike/aerospike-enterprise \
			 --set-file featureKeyFile=/secrets/aerospike/features.conf \
			 --set dbReplicas=4 \
			 --set externalIPServices.enabled=true \
			 --set externalIPServices.externalIPEndpoints[0].IP=10.160.15.224 \
			 --set externalIPServices.externalIPEndpoints[0].Port=7001 \
			 --set externalIPServices.externalIPEndpoints[1].IP=10.160.15.224 \
			 --set externalIPServices.externalIPEndpoints[1].Port=7002 \
			 --set externalIPServices.externalIPEndpoints[2].IP=10.160.15.223 \
			 --set externalIPServices.externalIPEndpoints[2].Port=8001 \
			 --set externalIPServices.externalIPEndpoints[3].IP=10.160.15.223 \
			 --set externalIPServices.externalIPEndpoints[3].Port=8002
```

For Helm v2,

```sh
helm install --name aerospike-release aerospike/aerospike-enterprise \
			 --set-file featureKeyFile=/secrets/aerospike/features.conf \
			 --set dbReplicas=4 \
			 --set externalIPServices.enabled=true \
			 --set externalIPServices.externalIPEndpoints[0].IP=10.160.15.224 \
			 --set externalIPServices.externalIPEndpoints[0].Port=7001 \
			 --set externalIPServices.externalIPEndpoints[1].IP=10.160.15.224 \
			 --set externalIPServices.externalIPEndpoints[1].Port=7002 \
			 --set externalIPServices.externalIPEndpoints[2].IP=10.160.15.223 \
			 --set externalIPServices.externalIPEndpoints[2].Port=8001 \
			 --set externalIPServices.externalIPEndpoints[3].IP=10.160.15.223 \
			 --set externalIPServices.externalIPEndpoints[3].Port=8002
```

### Aerospike Monitoring Stack

Aerospike Helm Chart provides Aerospike Monitoring Stack which includes an Aerospike prometheus exporter (sidecar), Prometheus statefulset, Grafana statefulset and Alertmanager statefulset.

### Deploy Aerospike Prometheus Exporter (only)
Aerospike Prometheus Exporter (sidecar) can be enabled by setting `enableAerospikePrometheusExporter` option to `true`.

```sh
helm install aerospike-release aerospike/aerospike-enterprise \
			 --set-file featureKeyFile=/secrets/aerospike/features.conf \
			 --set enableAerospikePrometheusExporter=true
```

For Helm v2,

```sh
helm install --name aerospike-release aerospike/aerospike-enterprise \
			 --set-file featureKeyFile=/secrets/aerospike/features.conf \
			 --set enableAerospikePrometheusExporter=true
```

### Deploy Complete Monitoring Stack
To deploy a complete monitoring stack which includes Prometheus, Grafana and Alertmanager, set `enableAerospikeMonitoring` option to `true`.

Note that, setting `enableAerospikeMonitoring` to `true` will automatically enable Aerospike Prometheus Exporter (sidecar).

```sh
helm install aerospike-release aerospike/aerospike-enterprise \
			 --set-file featureKeyFile=/secrets/aerospike/features.conf \
			 --set enableAerospikeMonitoring=true
```

For Helm v2,

```sh
helm install --name aerospike-release aerospike/aerospike-enterprise \
			 --set-file featureKeyFile=/secrets/aerospike/features.conf \
			 --set enableAerospikeMonitoring=true
```

Use option `--set-file prometheus.aerospikeAlertRulesFile` to add a custom aerospike alert rules configuration file.

> `prometheus.aerospikeAlertRulesFile` should be a file path on helm "client" machine (where the user is running 'helm install')

Aerospike alert rules file can also be passed in base64 encoded form. Use `prometheus.aerospikeAlertRulesFileBase64` configuration to specify base64 encoded string of the Aerospike alert rules file.

Use option `--set-file alertmanager.alertmanagerConfFile` to add an alertmanager configuration file.

> `alertmanager.alertmanagerConfFile` should be a file path on helm "client" machine (where the user is running 'helm install')

Alertmanager configuration file can also be passed in base64 encoded form. Use `alertmanager.alertmanagerConfFileBase64` configuration to specify base64 encoded string of the alertmanager configuration file.

Check the below [configuration section](#configuration) or [`values.yaml`](values.yaml) file for more details on configuration of the `Aerospike Prometheus Exporter`, `Prometheus`, `Grafana` and `Alertmanager`.

### Aerospike's Quiescence Feature

Aerospike Helm Chart (EE) uses Aerospike's ["Quiescence"](https://www.aerospike.com/docs/operations/manage/cluster_mng/quiescing_node/index.html) feature to perform smooth scale down, rolling upgrades and rolling restart across the cluster.

### Aerospike Cluster Security

Aerospike Helm Chart (EE) supports [Aerospike's Access Control](https://www.aerospike.com/docs/operations/configure/security/access-control/index.html).

To enable access control in aerospike cluster, set `security.enabled` to `true` and specify credentials of a user who has "user-admin" privilege (default `admin/admin`) using `security.username` and `security.password` options.

Aerospike helm chart automatically creates a new `sys-admin` user with credentials as given in `security.helmUsername` and `security.helmPassword` (default `helm_operator/helm_operator`) for management of the Aerospike cluster.

### Aerospike TLS (Transport Layer Security)

Use `tlsConfig` configuration to define TLS certificates. Multiple certificates can be defined. Each entry in `tlsConfig` must contain a `tlsName` (Subject/CN field or X509v3 Subject Alternative Name), `tlsCAFile` (CA certificate), `tlsCertFile` (server certificate) and a `tlsKeyFile` (key file contains private key associated with the certificate).

For `tlsCAFile`, `tlsCertFile` and `tlsKeyFile`, provide either base64 encoded version of certificates using format `"b64enc:<Base64EncodedStringOfCertificateFile>"` or provide file path to certificates which is accessible inside the container using format `"file:<FilePathToCertficateFile>"`.

```yaml
tlsConfig:
  - tlsName: "aerospike-client"
    tlsCAFile: "b64enc:LS0tLS1CRUdJTiBDCACERT"
    tlsCertFile: "b64enc:Q2VydGlmaWNhdGCERT"
    tlsKeyFile: "b64enc:LS0tLS1CRUdJTiBDKEY"
  - tlsName: "aerospike-cluster"
    tlsCAFile: "file:/secrets/certs/aerocacert.pem"
    tlsCertFile: "file:/secrets/certs/aerocert.pem"
    tlsKeyFile: "file:/secrets/certs/aerokey.pem"
```

If the certificates are available as a kubernetes secret, the secret can be mounted as a volume using `volumes` configuration and the file path to each certificate can be specified using `file:<FilePathToCertficateFile>` format.

```yaml
volumes:
 - mountPath: /secrets/certs/
   name: secret-dir
   template:
     secret:
      secretName: aerospike-certs
```

Use `aerospikeNetworkTLSConfig` to specify Aerospike network TLS configuration. `aerospikeNetworkTLSConfig` contains three configuration keys - `service`(client-to-cluster TLS), `heartbeat`(node-to-node TLS) and `fabric`(node-to-node TLS).

Each key further contains the following configurations,
- `tlsOnly` - enables only TLS traffic (disables non-TLS traffic)
- `tlsName` - TLS Name of one of the certificates defined in `tlsConfig`
- `tlsPort` - TLS port number

To enable mutual authentication in client-to-cluster communication, set `aerospikeNetworkTLSConfig.service.mutualAuthentication` to `true`.

Here's an example configuration that enables TLS for all three traffic - `service`, `heartbeat` and `fabric`.

```yaml
aerospikeNetworkTLSConfig:
  service:
    tlsOnly: true
    tlsName: aerospike-client
    tlsPort: 4333
    mutualAuthentication: true
  heartbeat:
    tlsOnly: true
    tlsName: aerospike-cluster
    tlsPort: 3012
  fabric:
    tlsOnly: true
    tlsName: aerospike-cluster
    tlsPort: 3011
```

### Configuration

| Parameter                                             | Description                                                                                                                                                                               | Default Value                                                                                                        |
| ------------------------------------------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|:--------------------------------------------------------------------------------------------------------------------:|
| `dbReplicas`                                          | Number of Aerospike nodes or pods in the cluster                                                                                                                                          | `3`                                                                                                                  |
| `terminationGracePeriodSeconds`                       | Number of seconds to wait after `SIGTERM` before force killing the pod.                                                                                                                   | `600`                                                                                                                |
| `livenessProbe`                                       | Configure livenessProbe for Aerospike container                                                                                                                                           | `initialDelaySeconds=30`, `periodSeconds=30`, rest - kubernetes defaults                                             |
| `readinessProbe`                                      | Configure readinessProbe for Aerospike container                                                                                                                                          | `initialDelaySeconds=30`, rest - kubernetes defaults                                                                 |
| `clusterServiceDnsDomain`                             | Kubernetes cluster service DNS domain                                                                                                                                                     | `cluster.local`                                                                                                      |
| `image.repository`                                    | Aerospike Server Docker Image                                                                                                                                                             | `aerospike/aerospike-server-enterprise`                                                                              |
| `image.tag`                                           | Aerospike Server Docker Image Tag                                                                                                                                                         | `5.5.0.7`                                                                                                            |
| `initImage.repository`                                | Aerospike Kubernetes Init Container Image                                                                                                                                                 | `aerospike/aerospike-kubernetes-init`                                                                                |
| `initImage.tag`                                       | Aerospike Kubernetes Init Container Image Tag                                                                                                                                             | `latest`                                                                                                             |
| `autoGenerateNodeIds`                                 | Auto generate and assign node-id(s) based on Pod's Ordinal Index                                                                                                                          | `true`                                                                                                               |
| `nodeIDPrefix`                                        | Node ID prefix                                                                                                                                                                            | `a`                                                                                                                  |
| `aerospikeNamespace`                                  | Aerospike Namespace name                                                                                                                                                                  | `test`                                                                                                               |
| `aerospikeNamespaceMemoryGB`                          | Aerospike Namespace Memory in GB                                                                                                                                                          | `1`                                                                                                                  |
| `aerospikeReplicationFactor`                          | Aerospike Namespace Replication Factor                                                                                                                                                    | `2`                                                                                                                  |
| `aerospikeDefaultTTL`                                 | Aerospike Namespace Record default TTL                                                                                                                                                    | `0` (Never Expire)                                                                                                   |
| `aerospikeFeatureKeyFilePath`                         | Aerospike `feature-key-file` path accessible from within the container (if using mounted volumes to share feature key file)                                                               | `not defined`                                                                                                        |
| `aerospikeClientPort`                                 | Aerospike TCP Service Port                                                                                                                                                                | `3000`                                                                                                               |
| `aerospikeHeartbeatPort`                              | Aerospike TCP Hearbeat Port                                                                                                                                                               | `3002`                                                                                                               |
| `aerospikeFabricPort`                                 | Aerospike TCP Fabric Port                                                                                                                                                                 | `3001`                                                                                                               |
| `aerospikeInfoPort`                                   | Aerospike TCP Info Port                                                                                                                                                                   | `3003`                                                                                                               |
| `security.enabled`                                    | Enable Aerosike Cluster Access Control                                                                                                                                                    | `false`                                                                                                              |
| `security.username`                                   | Specify User Admin Username                                                                                                                                                               | `admin`                                                                                                              |
| `security.password`                                   | Specify User Admin Password                                                                                                                                                               | `admin`                                                                                                              |
| `security.helmUsername`                               | Specify username for the `sys-admin` user to be created                                                                                                                                   | `helm_operator`                                                                                                      |
| `security.helmPassword`                               | Specify password for the `sys-admin` user to be created                                                                                                                                   | `helm_operator`                                                                                                      |
| `security.authMode`                                   | Auth mode for the access-control security                                                                                                                                                 | `internal`                                                                                                           |
| `tlsConfig`                                           | Define TLS certificates                                                                                                                                                                   | `[]`                                                                                                                 |
| `aerospikeNetworkTLSConfig`                           | Define Aerospike network TLS configurations                                                                                                                                               | `{}`                                                                                                                 |
| `args`                                                | Define additional arguments to be passed to the Aerospike container                                                                                                                       | `[]`                                                                                                                 |
| `autoRolloutConfig`		   	                        | Rollout ConfigMap/Secrets changes on 'helm upgrade'    			                                                                                                                        | `false`					   	                                                                                       |
| `hostNetwork.enabled`		 			                | Enable `hostNetwork`. Allows Pods to access host network.			                                                                                                                        | `false`					   	                                                                                       |
| `hostNetwork.useExternalIP`		 			        | Allow applications to connect using external IP of the instances			                                                                                                                | `false`					   	                                                                                       |
| `nodePortServices.enabled`		 	                | Enable NodePort Services (`Type: NodePort`) to expose aerospike statefulset                                                                                                               | `false`					   	                                                                                       |
| `nodePortServices.useExternalIP`		 			    | Allow applications to connect using external IP of the instances			                                                                                                                | `false`					   	                                                                                       |
| `loadBalancerServices.enabled`		                | Enable LoadBalancer Services (`Type: LoadBalancer`) to expose aerospike statefulset                                                                                                       | `false`					                                                                                           |
| `externalIPServices.enabled`		                    | Enable external IP Services (`Type: ClusterIP`) to expose aerospike statefulset                                                                                                           | `false`					                                                                                           |
| `externalIPServices.externalIPEndpoints`		 		| Specify External IP/Port endpoints for `externalIPServices`	                                                                                                                            | `[]` (nil)					   	                                                                                   |
| `rbac.create`		 			                        | Create a new ServiceAccount with a new ClusterRole for Aerospike Statefulset                                                                                                              | `true`					   	                                                                                       |
| `rbac.serviceAccountName`		 	                    | Specify an existing ServiceAccount to use with Aerospike Statefulset                                                                                                                      | `default`					   	                                                                                       |
| `antiAffinity`		 			                    | Enable `PodAntiAffinity` rule to schedule one Aerospike pod per node. Supported values - `off`, `soft`, `hard`                                                                            | `off`                                                                                                                |
| `antiAffinityWeight`		 		                    | 'weight' in range 1-100 for "soft" antiAffinity option for Aerospike pods    			                                                                                                    | `1`					   		                                                                                       |
| `affinity`		 				                    | Define custom `nodeAffinity`/`podAffinity`/`podAntiAffinity` rules for Aerospike pods	                                                                                                    | `{}` (nil)				   	                                                                                       |
| `tolerations`		 				                    | Define tolerations for scheduling Aerospike pods based on node taints                                                                                                                     | `[]` (nil)				   	                                                                                       |
| `nodeSelector`	 				                    | Define nodeSelector for scheduling Aerospike pods based on node labels                                                                                                                    | `{}` (nil)				   	                                                                                       |
| `labels`	 				                            | Define labels for Aerospike StatefulSet                                                                                                                                                   | `{}` (nil)				   	                                                                                       |
| `annotations`	 				                        | Define annotations for Aerospike StatefulSet                                                                                                                                              | `{}` (nil)				   	                                                                                       |
| `podLabels`	 				                        | Define labels for Aerospike pods                                                                                                                                                          | `{}` (nil)				   	                                                                                       |
| `podAnnotations`	 				                    | Define annotations for Aerospike pods                                                                                                                                                     | `{}` (nil)                                                                                                           |
| `persistenceStorage`                                  | Define Peristent Volumes to be used (Map - to define multiple volumes)                                                                                                                    | `{}` (nil)                                                                                                           |
| `volumes`                                             | Define volumes section and template to be used                                                                                                                                            | `volume[0].mountPath: /opt/aerospike/data`,<br />`volume[0].name: datadir`,<br />`volume[0].template: emptyDir: {}`  |
| `resources`                                           | Resource configuration (`requests` and `limits`)                                                                                                                                          | `{}` (nil)                                                                                                           |
| `podSecurityContext`                                  | Aerospike pod security context                                                                                                                                                            | `{}` (nil)                                                                                                           |
| `securityContext`                                     | Aerospike container security context                                                                                                                                                      | `{}` (nil)                                                                                                           |
| `aerospikeConfFile`                                   | Custom aerospike.conf file path on helm client machine (To be used during the runtime, `helm install` .. etc)                                                                             | `not defined`                                                                                                        |
| `aerospikeConfFileBase64`                             | Custom Aerospike configuration file as base64 encoded string                                                                                                                              | `"" (not defined)`                                                                                                   |
| `featureKeyFile`                                      | Feature Key File (Enterprise License) file location on helm client machine (To be used during the runtime, `helm install` .. etc)                                                         | `not defined`                                                                                                        |
| `featureKeyFileBase64`                                | Feature key file as base64 encoded string                                                                                                                                                 | `"" (not defined)`                                                                                                   |
| `prometheus.aerospikeAlertRulesFile`                  | Aerospike alert rules configuration file location on helm client machine (To be used during the runtime, `helm install` .. etc)                                                           | `not defined`                                                                                                        |
| `prometheus.aerospikeAlertRulesFileBase64`            | Aerospike alert rules file as base64 encoded string                                                                                                                                       | `"" (not defined)`                                                                                                   |
| `alertmanager.alertmanagerConfFile`                   | Alertmanager configuration file location on helm client machine (To be used during the runtime, `helm install` .. etc)                                                                    | `not defined`                                                                                                        |
| `alertmanager.alertmanagerConfFileBase64`             | Alertmanager configuration file as base64 encoded string                                                                                                                                  | `"" (not defined)`                                                                                                   |
| `enableAerospikePrometheusExporter`	                | Enable Sidecar Aerospike Prometheus Exporter (only)                                                                                                                                       | `false`					   	                                                                                       |
| `enableAerospikeMonitoring`		 	                | Enable Aerospike Monitoring - sidecar prometheus exporter, Prometheus, Grafana, Alertmanager stack                                                                                        | `false`					   	                                                                                       |
| `exporter.repository`                                 | Aerospike prometheus exporter image repository                                                                                                                                            | `aerospike/aerospike-prometheus-exporter`                                                                            |
| `exporter.tag`                                        | Aerospike prometheus exporter image tag                                                                                                                                                   | `latest`                                                                                                             |
| `exporter.agentCertFile`                              | Certificate file for TLS between exporter and prometheus server                                                                                                                           | `"" (not defined)`                                                                                                   |
| `exporter.agentKeyFile`                               | Key file for TLS between exporter and Prometheus server                                                                                                                                   | `"" (not defined)`                                                                                                   |
| `exporter.metricLabels`                               | Aerospike prometheus exporter custom labels for metrics                                                                                                                                   | `""`                                                                                                                 |
| `exporter.agentBindHost`                              | Aerospike prometheus exporter IP to bind to                                                                                                                                               | `""`                                                                                                                 |
| `exporter.agentBindPort`                              | Aerospike prometheus exporter port to bind to                                                                                                                                             | `9145`                                                                                                               |
| `exporter.agentTimeout`                               | Metrics server timeout (in seconds)                                                                                                                                                       | `10`                                                                                                                 |
| `exporter.agentLogFile`                               | Aerospike prometheus exporter log file path                                                                                                                                               | `"" (not defined)`                                                                                                   |
| `exporter.agentLogLevel`                              | Aerospike prometheus exporter logging level                                                                                                                                               | `"info"`                                                                                                             |
| `exporter.httpBasicAuthUsername`                      | Basic HTTP Authentication username for `/metrics` URL of the aerospike prometheus exporter                                                                                                | `"" (not defined)`                                                                                                   |
| `exporter.httpBasicAuthPassword`                      | Basic HTTP Authentication password for `/metrics` URL of the aerospike prometheus exporter                                                                                                | `"" (not defined)`                                                                                                   |
| `exporter.aerospikeHost`                              | Aerospike container service IP to connect to                                                                                                                                              | `"localhost"`                                                                                                        |
| `exporter.aerospikePort`                              | Aerospike container service port to connect to                                                                                                                                            | `3000`                                                                                                               |
| `exporter.infoTimeout`                                | Timeout (in seconds) for sending info commands to aerospike container                                                                                                                     | `5`                                                                                                                  |
| `exporter.namespaceMetricsAllowlist`                  | Namespace metrics allowlist for aerospike prometheus exporter                                                                                                                             | `not defined`                                                                                                        |
| `exporter.setMetricsAllowlist`                        | Set metrics allowlist for aerospike prometheus exporter                                                                                                                                   | `not defined`                                                                                                        |
| `exporter.nodeMetricsAllowlist`                       | Node metrics allowlist for aerospike prometheus exporter                                                                                                                                  | `not defined`                                                                                                        |
| `exporter.xdrMetricsAllowlist`                        | XDR metrics allowlist for aerospike prometheus exporter                                                                                                                                   | `not defined`                                                                                                        |
| `exporter.namespaceMetricsBlocklist`                  | Namespace metrics blocklist for aerospike prometheus exporter                                                                                                                             | `not defined`                                                                                                        |
| `exporter.setMetricsBlocklist`                        | Set metrics blocklist for aerospike prometheus exporter                                                                                                                                   | `not defined`                                                                                                        |
| `exporter.nodeMetricsBlocklist`                       | Node metrics blocklist for aerospike prometheus exporter                                                                                                                                  | `not defined`                                                                                                        |
| `exporter.xdrMetricsBlocklist`                        | XDR metrics blocklist for aerospike prometheus exporter                                                                                                                                   | `not defined`                                                                                                        |
| `prometheus.replicas`                                 | Number of replicas for prometheus statefulset                                                                                                                                             | `2`                                                                                                                  |
| `prometheus.serverPort`                               | Prometheus server port                                                                                                                                                                    | `9090`                                                                                                               |
| `prometheus.terminationGracePeriodSeconds`            | Number of seconds to wait after `SIGTERM` before force killing the pod.                                                                                                                   | `120`                                                                                                                |
| `prometheus.livenessProbe`                            | Configure livenessProbe for Prometheus container                                                                                                                                          | `initialDelaySeconds=30`, `timeoutSeconds=10`, rest - kubernetes defaults                                            |
| `prometheus.readinessProbe`                           | Configure readinessProbe for Prometheus container                                                                                                                                         | `initialDelaySeconds=30`, `timeoutSeconds=10`, rest - kubernetes defaults                                            |
| `prometheus.scrapeInterval`                           | Prometheus `scrape_interval` in seconds to define interval between each scrape                                                                                                            | `15s`                                                                                                                |
| `prometheus.evaluationInterval`                       | How frequently to evaluate alert rules                                                                                                                                                    | `15s`                                                                                                                |
| `prometheus.image.repository`                         | Prometheus Docker Image Repository                                                                                                                                                        | `prom/prometheus`                                                                                                    |
| `prometheus.image.tag`                                | Prometheus Docker Image Tag                                                                                                                                                               | `latest`                                                                                                             |
| `prometheus.persistenceStorage`                       | Define storage for prometheus data                                                                                                                                                        | `{}` (nil)                                                                                                           |
| `prometheus.volume`                                   | Define storage for prometheus data                                                                                                                                                        | `volume.mountPath: /data`,<br />`volume.name: prometheus-data`,<br />`volume.template: emptyDir: {}`                 |
| `prometheus.resources`                                | Resource configuration (`requests` and `limits`)                                                                                                                                          | `{}` (nil)                                                                                                           |
| `prometheus.tolerations`		 				        | Define tolerations for scheduling prometheus pods based on node taints                                                                                                                    | `[]` (nil)				   	                                                                                       |
| `prometheus.nodeSelector`	 				            | Define nodeSelector for scheduling prometheus pods based on node labels                                                                                                                   | `{}` (nil)				   	                                                                                       |
| `prometheus.antiAffinity`		 			            | Enable `PodAntiAffinity` rule to schedule one prometheus pod per node. Supported values - `off`, `soft`, `hard`                                                                           | `off`                                                                                                                |
| `prometheus.antiAffinityWeight`		 		        | 'weight' in range 1-100 for "soft" antiAffinity option for prometheus pods   			                                                                                                    | `1`					   		                                                                                       |
| `prometheus.affinity`		 				            | Define custom `nodeAffinity`/`podAffinity`/`podAntiAffinity` rules for prometheus pods	                                                                                                | `{}` (nil)				   	                                                                                       |
| `grafana.replicas`                                    | Number of replicas for grafana statefulset                                                                                                                                                | `1`                                                                                                                  |
| `grafana.httpPort`                                    | Grafana server `http_port`                                                                                                                                                                | `3000`                                                                                                               |
| `grafana.plugins`                                     | Grafana plugins to install at startup                                                                                                                                                     | `"camptocamp-prometheus-alertmanager-datasource"`                                                                    |
| `grafana.image.repository`                            | Grafana Docker Image Repository                                                                                                                                                           | `grafana/grafana`                                                                                                    |
| `grafana.image.tag`                                   | Grafana Docker Image Tag                                                                                                                                                                  | `latest`                                                                                                             |
| `grafana.terminationGracePeriodSeconds`               | Number of seconds to wait after `SIGTERM` before force killing the pod.                                                                                                                   | `120`                                                                                                                |
| `grafana.livenessProbe`                               | Configure livenessProbe for Grafana container                                                                                                                                             | `initialDelaySeconds=30`, `timeoutSeconds=10`, `failureThreshold=10`, rest - kubernetes defaults                     |
| `grafana.readinessProbe`                              | Configure readinessProbe for Grafana container                                                                                                                                            | `initialDelaySeconds=30`, `timeoutSeconds=10`, `failureThreshold=10`, rest - kubernetes defaults                     |
| `grafana.persistenceStorage`                          | Define storage for grafana data                                                                                                                                                           | `{}` (nil)                                                                                                           |
| `grafana.volume`                                      | Define storage for grafana data                                                                                                                                                           | `volume.mountPath: /var/lib/grafana`,<br />`volume.name: grafana-data`,<br />`volume.template: emptyDir: {}`         |
| `grafana.resources`                                   | Resource configuration (`requests` and `limits`)                                                                                                                                          | `{}` (nil)                                                                                                           |
| `grafana.tolerations`		 				            | Define tolerations for scheduling grafana pods based on node taints                                                                                                                       | `[]` (nil)				   	                                                                                       |
| `grafana.nodeSelector`	 				            | Define nodeSelector for scheduling grafana pods based on node labels                                                                                                                      | `{}` (nil)				   	                                                                                       |
| `grafana.antiAffinity`		 			            | Enable `PodAntiAffinity` rule to schedule one grafana pod per node. Supported values - `off`, `soft`, `hard`                                                                              | `off`                                                                                                                |
| `grafana.antiAffinityWeight`		 		            | 'weight' in range 1-100 for "soft" antiAffinity option for grafana pods   			                                                                                                    | `1`					   		                                                                                       |
| `grafana.affinity`		 				            | Define custom `nodeAffinity`/`podAffinity`/`podAntiAffinity` rules for grafana pods	                                                                                                    | `{}` (nil)				   	                                                                                       |
| `grafana.user`                                        | Grafana username and password                                                                                                                                                             | `"admin"`                                                                                                            |
| `grafana.password`                                    | Grafana username and password                                                                                                                                                             | `"admin"`                                                                                                            |
| `alertmanager.replicas`                               | Number of replicas for alertmanager statefulset                                                                                                                                           | `1`                                                                                                                  |
| `alertmanager.webPort`                                | Alertmanager web port                                                                                                                                                                     | `9093`                                                                                                               |
| `alertmanager.meshPort`                               | Alertmanager gossip port                                                                                                                                                                  | `9094`                                                                                                               |
| `alertmanager.terminationGracePeriodSeconds`          | Number of seconds to wait after `SIGTERM` before force killing the pod.                                                                                                                   | `120`                                                                                                                |
| `alertmanager.livenessProbe`                          | Configure livenessProbe for Alertmanager container                                                                                                                                        | `initialDelaySeconds=30`, `timeoutSeconds=10`, rest - kubernetes defaults                                            |
| `alertmanager.readinessProbe`                         | Configure readinessProbe for Alertmanager container                                                                                                                                       | `initialDelaySeconds=30`, `timeoutSeconds=10`, rest - kubernetes defaults                                            |
| `alertmanager.image.repository`                       | Alertmanager Docker Image Repository                                                                                                                                                      | `prom/alertmanager`                                                                                                  |
| `alertmanager.image.tag`                              | Alertmanager Docker Image Tag                                                                                                                                                             | `latest`                                                                                                             |
| `alertmanager.loglevel`                               | Alertmanager logging level                                                                                                                                                                | `info`                                                                                                               |
| `alertmanager.persistenceStorage`                     | Define storage for alertmanager data                                                                                                                                                      | `{}` (nil)                                                                                                           |
| `alertmanager.volume`                                 | Define storage for alertmanager data                                                                                                                                                      | `volume.mountPath: /data`,<br />`volume.name: alertmanager-data`,<br />`volume.template: emptyDir: {}`               |
| `alertmanager.resources`                              | Resource configuration (`requests` and `limits`)                                                                                                                                          | `{}` (nil)                                                                                                           |
| `alertmanager.tolerations`		                    | Define tolerations for scheduling alertmanager pods based on node taints                                                                                                                  | `[]` (nil)				   	                                                                                       |
| `alertmanager.nodeSelector`	 	                    | Define nodeSelector for scheduling alertmanager pods based on node labels                                                                                                                 | `{}` (nil)				   	                                                                                       |
| `alertmanager.antiAffinity`		                    | Enable `PodAntiAffinity` rule to schedule one alertmanager pod per node. Supported values - `off`, `soft`, `hard`                                                                         | `off`                                                                                                                |
| `alertmanager.antiAffinityWeight`	                    | 'weight' in range 1-100 for "soft" antiAffinity option for alertmanager pods   			                                                                                                | `1`					   		                                                                                       |
| `alertmanager.affinity`		 	                    | Define custom `nodeAffinity`/`podAffinity`/`podAntiAffinity` rules for alertmanager pods	                                                                                                | `{}` (nil)				   	                                                                                       |


Note that the namespace related configurations (`aerospikeNamespace`, `aerospikeNamespaceMemoryGB`, `aerospikeReplicationFactor` and `aerospikeDefaultTTL`) are intended for default single namespace configuration.

If using multiple namespaces, these config items can be ignored and a separate `aerospike.conf` file or template with multiple namespace configuration can be used.
