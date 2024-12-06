# helm-charts

Qpoint Helm charts are available at <https://helm.qpoint.io/>.

```sh
helm repo add qpoint https://helm.qpoint.io/
"qpoint" has been added to your repositories
```

Then run the following to see a list of available charts:

```sh
helm search repo qpoint
```

## Releasing a new version

 1. Create a PR that updates `Chart.yaml`'s version numbers.

```yaml
# This is the chart version. This version number should be incremented each time you make changes
# to the chart and its templates, including the app version.
# Versions are expected to follow Semantic Versioning (https://semver.org/)
version: 0.0.18 # 🚨 Update this version number

# This is the version number of the application being deployed. This version number should be
# incremented each time you make changes to the application. Versions are not expected to
# follow Semantic Versioning. They should reflect the version the application is using.
# It is recommended to use it with quotes.
appVersion: "v0.5.14" # 🚨 Update this version number
```

 2. Merge the PR and the index.yaml file will be updated automatically and deployed to the helm repo.
