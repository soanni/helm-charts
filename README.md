# Helm Charts

A small collection of Helm charts, served as a Helm repository from GitHub Pages.

**Repository URL:** `https://soanni.github.io/helm-charts`

## Add the repository

```bash
helm repo add soanni https://soanni.github.io/helm-charts
helm repo update
```

List the available charts:

```bash
helm search repo soanni
```

## Charts

### `raw`

Renders arbitrary raw Kubernetes manifests from values. Useful when you need to
ship one or more plain manifests (a `ConfigMap`, `Secret`, `Ingress`, CRD, etc.)
through Helm — for example as a subchart dependency, or to bundle extra objects
alongside another release — without writing a full chart of your own.

Each entry under `extraObjects` is a manifest that is passed through Helm's
[`tpl`](https://helm.sh/docs/howto/charts_tips_and_tricks/#using-the-tpl-function)
function, so you can use template expressions and reference values inside them.

#### Install

```bash
helm install my-objects soanni/raw -f values.yaml
```

#### Example `values.yaml`

```yaml
extraObjects:
  - apiVersion: v1
    kind: ConfigMap
    metadata:
      name: app-config
    data:
      greeting: "hello"

  - apiVersion: v1
    kind: Secret
    metadata:
      name: app-secret
    type: Opaque
    stringData:
      # Templating is supported via tpl. `.Release` and other built-ins work.
      release: "{{ .Release.Name }}"
```

#### Values

| Key            | Type   | Default | Description                                              |
| -------------- | ------ | ------- | -------------------------------------------------------- |
| `extraObjects` | list   | `[]`    | List of Kubernetes manifests to render. Each is run through `tpl`. |

#### Use as a subchart

Add it as a dependency in another chart's `Chart.yaml`:

```yaml
dependencies:
  - name: raw
    version: 0.1.1
    repository: https://soanni.github.io/helm-charts
```

Then define objects under the `raw` key in the parent chart's values:

```yaml
raw:
  extraObjects:
    - apiVersion: v1
      kind: ConfigMap
      metadata:
        name: extra
      data:
        key: value
```

## Preview the rendered output

Before installing, you can see exactly what will be applied:

```bash
helm template my-objects soanni/raw -f values.yaml
```

## Uninstall

```bash
helm uninstall my-objects
```

## Development

Charts live under `charts/`. To work on a chart locally:

```bash
# Lint
helm lint charts/raw

# Render with a local values file
helm template test charts/raw -f my-values.yaml
```

Releases are automated: on every push to `main`, the
[chart-releaser](https://github.com/helm/chart-releaser-action) GitHub Action
packages any chart whose `version` in `Chart.yaml` has changed, creates a
GitHub release, and updates the Helm repository index on the `gh-pages` branch.

**To publish a new version, bump `version` in the chart's `Chart.yaml` and push to `main`.**

## License

See [LICENSE](LICENSE).
