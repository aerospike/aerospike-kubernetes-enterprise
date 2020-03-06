# Change Log

This file documents all notable changes to Aerospike Helm Chart (Enterprise Edition).

## [1.2.0](https://github.com/aerospike/aerospike-kubernetes-enterprise/releases/tag/1.2.0)

- Uses new `aerospike/aerospike-kubernetes-init` image
- Aerospike tcp ports now configurable
- Improved security implementation
- Added support for NodePort type services to expose aerospike statefulset.
- Added support for LoadBalancer type services to expose aerospike statefulset.
- Added support for externalIP clusterIP type services to expose aerospike statefulset.
- Added configuration to specify or create serviceAccounts to be used in Aerospike/Prometheus/Grafana/Alertmanager statefulsets.
- Integrated Aerospike Monitoring stack with aerospike-prometheus-exporter, prometheus, grafana, and alertmanager.
- Added dynamic configuration to pass in Aerospike alert rules conf file and alertmanager conf file.
- Added support for Aerospike quiesce feature.
- Honor only `.Release.Namespace`. Removed `namespace` option from `values.yaml`
- Added `aerospike-prometheus-exporter` as `sidecar` container (applicable only when aerospike monitoring is enabled).
- Chart `4.6.0` updated to use Aerospike Server version `4.6.0.12`
- Chart `4.7.0` updated to use Aerospike Server version `4.7.0.10`
- Chart `4.8.0` updated to use Aerospike Server version `4.8.0.5`


## [1.1.0](https://github.com/aerospike/aerospike-kubernetes-enterprise/releases/tag/1.1.0)
- Update Chart `4.7.0` to use Aerospike Server version `4.7.0.5` (appVersion).
- Update Chart `4.6.0` to use Aerospike Server version `4.6.0.8` (appVersion).

## [1.0.0](https://github.com/aerospike/aerospike-kubernetes-enterprise/releases/tag/1.0.0)

- Supports `NodeAffinity`/`PodAffinity`/`PodAntiAffinity` rules.
   - Set `antiAffinity` to ensure one Pod per Node (for a release).
     Supported Values:  `off`, `soft` ('preferred' during scheduling), and `hard` ('required' during scheduling). Default : `off`
   - Set `antiAffinityWeight` option to specify 'weight' for 'soft' 'antiAffinity' option above. Default : `1`
   - Users can also define their custom `PodAffinity`/`PodAntiAffinity`/`NodeAffinity` rules using a third option `affinity`. Default: `{}` (not set)
- Auto rollout changes to ConfigMaps on helm upgrade.
   - Set `autoRolloutConfig=true`. Default: `false`
- Added option `autoGenerateNodeIds` to generate unique default node-ids. Default: `false`
- Added option `hostNetworking` to enable host networking. Default: `false`
- Added option `platform` to work with hostNetworking. Use both to auto-configure Aerospike to use external IP as alternate access address if it exists.
Supported values : `none`, `gke`, and `eks`
- Peer finder will now work with hostNetworking and use K8s Cluster DNS.
- Renamed `dBReplicas` to `dbReplicas`.
- Increased termination grace period from `30` to `120` default.
- Changed default `aerospikeDefaultTTL` to `0` (Never Expire), dbReplicas to `3`.
- Update Chart `4.7.0` to use Aerospike Server version `4.7.0.2` (appVersion).
- Update Chart `4.6.0` to use Aerospike Server version `4.6.0.5` (appVersion).