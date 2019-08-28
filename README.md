# aerospike-kubernetes-enterprise

This project contains the init container used in Kubernetes (k8s) and the Aerospike StatefulSet definition.
This manifest will allow you to deploy a fully formed Aerospike cluster in minutes.

This project uses:

- [aerospike-server-enterprise docker image](https://hub.docker.com/r/aerospike/aerospike-server-enterprise)
- [aerospike-tools docker image](https://hub.docker.com/r/aerospike/aerospike-tools)

## Usage:

### Configure:

Set environment variables (modify if necessary):

```sh
export APP_NAME=aerospike
export NAMESPACE=default
export AEROSPIKE_NODES=3
export AEROSPIKE_NAMESPACE=test
export AEROSPIKE_REPL=2
export AEROSPIKE_MEM=1
export AEROSPIKE_TTL=0
export AEROSPIKE_FEATURE_KEY_FILE=/etc/aerospike/features.conf
```

All `AEROSPIKE_*` parameters except AEROSPIKE\_NODES, AEROSPIKE_MEM are optional. Default values are listed above.
All other parameters are required.

**NOTE:** Feature key file is mandatory for running aerospike server enterprise edition.


### Configuring Storage:

The [statefulset definition](manifests/statefulset.yaml) refers to a custom StorageClass `ssd`.
You can find the storageclass `ssd` definition in [storageclass-aws.yaml](manifests/storageclass-aws.yaml) or [storageclass-gcp.yaml](manifests/storageclass-gcp.yaml) (Uncomment them to use). You can also define your own storageclass and use it within the statefulset definition.

If you want to use the raw block volume mode, you need to define `volumeMode` as `Block` in the Volume Claim and use `volumeDevices` and `devicePath` instead of `volumeMounts` and `mountPath` as shown in the example below.

```
  volumeClaimTemplates:
  - metadata:
      name: data-dev
      labels: *AerospikeDeploymentLabels
    spec:
      volumeMode: Block
      accessModes:
        - ReadWriteOnce
      storageClassName: ssd
      resources:
        requests:
          storage: ${AEROSPIKE_MEM}Gi
```
```
.....
volumeMounts:
        - name: confdir
          mountPath: /etc/aerospike
volumeDevices:
        - name: data-dev
          devicePath: /dev/sdb
.....
```

For Kubernetes version > 1.11, there's a default storage class `gp2` available on AWS EKS clusters, uses `aws-ebs` provisioner and volume type `gp2`.

### Apply feature-key-file:

To apply feature-key-file, simply add the file to `configs/` and create the ConfigMap. If using mounted volumes to apply the feature-key-file, you can use `AEROSPIKE_FEATURE_KEY_FILE` to specify the file path within the container.


### Examples:

To view and run the examples, go to [`examples/`](examples/)

### Deployment:

Please follow the below steps or run `start.sh` script:

1. Expand manifest template:

```sh
cat manifests/* | envsubst > expanded.yaml
```

2. Create the configmap object:

```sh
kubectl create configmap aerospike-conf -n $NAMESPACE --from-file=configs/
```

3. Deploy:

```sh
kubectl create -f expanded.yaml
```

### Helm Charts

Helm chart for the same can be found [here](helm/)

## Requirements

* Kubernetes 1.8+
* Kubernetes DNS add-in
