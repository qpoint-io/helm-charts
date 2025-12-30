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

## Configuration

The following table lists the configurable parameters of the Qpoint TAP chart and their default values.

| Parameter | Description | Default | Allowed Values |
|-----------|-------------|---------|----------------|
| `image.repository` | Container image repository | `us-docker.pkg.dev/qpoint-edge/public/qtap` | Valid container registry path |
| `image.pullPolicy` | Container image pull policy | `IfNotPresent` | `IfNotPresent`, `Always`, `Never` |
| `image.tag` | Container image tag | `""` (defaults to chart appVersion) | Valid image tag |
| `imagePullSecrets` | Image pull secrets | `[]` | List of secret names |
| `nameOverride` | Override the name of the chart | `""` | String |
| `fullnameOverride` | Override the full name of the chart | `""` | String |
| `args` | Container arguments | `[]` | List of strings |
| `registrationEndpoint` | Qpoint API registration endpoint | `https://api.qpoint.io` | Valid URL |
| `registrationToken` | API token for registration | `""` | Valid token string |
| `registrationTokenSecretRefName` | Name of secret containing registration token. Overrides `registrationToken` | `""` | Valid Kubernetes secret name |
| `serviceAccount.create` | Create a service account | `true` | `true`, `false` |
| `serviceAccount.automount` | Automatically mount API credentials | `true` | `true`, `false` |
| `serviceAccount.annotations` | Annotations for the service account | `{}` | Valid Kubernetes annotations |
| `serviceAccount.name` | Name of the service account | `""` | Valid service account name |
| `podAnnotations` | Annotations for pods | `{}` | Valid Kubernetes annotations |
| `podLabels` | Additional labels for pods | `{}` | Valid Kubernetes labels |
| `podSecurityContext` | Security context for pods | `{}` | Valid pod security context |
| `securityContext.allowPrivilegeEscalation` | Allow privilege escalation | `true` | `true`, `false` |
| `securityContext.capabilities.add` | Add capabilities | `["CAP_BPF", "CAP_SYS_ADMIN"]` | List of Linux capabilities |
| `securityContext.readOnlyRootFilesystem` | Read-only root filesystem | `false` | `true`, `false` |
| `securityContext.runAsNonRoot` | Run as non-root user | `false` | `true`, `false` |
| `securityContext.runAsUser` | User ID to run as | `0` | Integer |
| `securityContext.runAsGroup` | Group ID to run as | `0` | Integer |
| `securityContext.privileged` | Run container in privileged mode | `true` | `true`, `false` |
| `logLevel` | Logging level | `info` | `debug`, `info`, `warn`, `error`, `dpanic`, `panic`, `fatal` |
| `logEncoding` | Log output format | `json` | `console`, `json` |
| `status.addr` | Status server address | `0.0.0.0` | Valid IP address |
| `status.port` | Status server port | `10001` | Valid port number |
| `middlewareEgress.addr` | Middleware egress address | `127.0.0.1` | Valid IP address |
| `middlewareEgress.port` | Middleware egress port | `11001` | Valid port number |
| `resources.limits.cpu` | CPU resource limit | `1000m` | Valid Kubernetes CPU resource |
| `resources.limits.memory` | Memory resource limit | `1Gi` | Valid Kubernetes memory resource |
| `resources.requests.cpu` | CPU resource request | `100m` | Valid Kubernetes CPU resource |
| `resources.requests.memory` | Memory resource request | `128Mi` | Valid Kubernetes memory resource |
| `volumes` | Additional volumes | See values.yaml | List of volume definitions |
| `volumeMounts` | Additional volume mounts | See values.yaml | List of volume mount definitions |
| `integrations.containerd.enabled` | Enable containerd integration | `true` | `true`, `false` |
| `integrations.containerd.hostSocketPath` | Path to containerd socket on host | `/run/containerd/containerd.sock` | Valid file path |
| `nodeSelector` | Node labels for pod assignment | `{}` | Valid node selector |
| `tolerations` | Tolerations for pod assignment | `[]` | List of tolerations |
| `affinity` | Affinity rules for pod assignment | `{}` | Valid affinity rules |
| `config` | TAP configuration | `""` | Valid YAML configuration |
| `extraEnv` | Additional environment variables | `[]` | List of environment variable definitions |

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
