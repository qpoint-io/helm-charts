# qpoint-connect Helm chart

This deploys Qpoint Proxy to Kubernetes using [Helm](https://helm.sh/).

The Qpoint Proxy can operate in two modes:

1. Control plane mode. For this mode it depends on a secret called `token` existing within the namespace that the chart is installed. Below is an example of how to create the secret before chart install:

```sh
kubectl --namespace qpoint create secret generic token --from-literal=token="<TOKEN>"
helm install qpoint-gateway . --namespace qpoint --create-namespace --wait --timeout 20s
```

2. Standalone mode. For this mode it depends on a config map named `config` with the file contents of a valid YAML configuration file format.

Both modes will deploy Qpoint with the `proxy` command. Any of the values can be overridden using standard Helm processes.
