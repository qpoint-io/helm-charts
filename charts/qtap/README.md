# qtap Helm chart

This deploys Qtap to Kubernetes using [Helm](https://helm.sh/).

It depends on a secret called `token` existing within the namespace that the chart is installed. Below is an example of how to create the secret before chart install:

```sh
kubectl --namespace qtap create secret generic token --from-literal=token="<TOKEN>"
helm install qtap-gateway . --namespace qtap --create-namespace --wait --timeout 20s
```

This will deploy Qtap with the `gateway` command. Any of the values can be overridden using standard Helm processes.

## Releasing the chart

Our chart is released under https://github.com/qpoint-io/helm-charts which is a GitHub page at <https://qpoint-io.github.io/helm-charts/>.

This chart is released with the following process:

```sh
helm package . -d ./packages/

```

The above will put the `.tgz` file within the `.packages` directory. This needs to be added as a release to the chart repository under https://github.com/qpoint-io/helm-charts/releases/new with the format `qtap-<VERSION>`. The following command should be ran to produce the `index.yaml`.

```sh
helm repo index --url https://qpoint-io.github.io/helm-charts/ .
```

The content of the produced `index.yaml` needs to be merged with https://github.com/qpoint-io/helm-charts/blob/main/index.yaml. Note that the `urls` entry needs to be updated with the link to the download for the release.
