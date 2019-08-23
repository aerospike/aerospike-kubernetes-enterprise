# aerospike-kubernetes-enterprise

This project contains the init container used in Kubernetes (k8s) and the Aerospike StatefulSet definition.

This manifest will allow you to deploy a fully formed Aerospike cluster in minutes.

Uses:
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


### Storage

You can configure your own storage class or uncomment either the `storageclass-gcp.yaml` or `storageclass-aws.yaml`.


### Apply feature-key-file

To apply feature-key-file, simply add the file to `configs/` and create the ConfigMap. If using mounted volumes to apply the feature-key-file, you can use the AEROSPIKE_FEATURE_KEY_FILE to specify the path.


### Deploy:

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


## Requirements

* Kubernetes 1.8+
* Kubernetes DNS add-in
