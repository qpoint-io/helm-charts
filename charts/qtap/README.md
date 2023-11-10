# qtap Helm chart

This deploys Qtap to Kubernetes using [Helm](https://helm.sh/).

It depends on a secret called `token` existing within the namespace that the chart is installed. Below is an example of how to create the secret before chart install:

```sh
kubectl --namespace qtap create secret generic token --from-literal=token="<TOKEN>"
helm install qtap-gateway . --namespace qtap --create-namespace --wait --timeout 20s
```

This will deploy Qtap with the `gateway` command. Any of the values can be overridden using standard Helm processes.
