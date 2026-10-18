# PostgreSQL Helm Chart

This folder contains a Helm chart for deploying PostgreSQL resources to a Kubernetes cluster.

## Folder structure

```text
helm/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── 0-namespace...
    ├── deployment...
    └── service.yml
```

> **Note:** Helm normally expects the chart metadata file to be named `Chart.yaml` and the default values file to be named `values.yaml`. If your files are actually named `chart.yml` and `values.yml`, rename them to these standard names so Helm recognizes them automatically.

## What is Helm?

Helm is a package manager for Kubernetes. A Helm chart contains templates and configuration values that Helm combines to generate Kubernetes manifests.

- **Chart.yaml** — chart name, version, and metadata.
- **values.yaml** — default configuration values.
- **templates/** — Kubernetes YAML templates rendered by Helm.
- **Release** — an installed instance of a chart in a cluster.

## How the templates work

The `templates/service.yml` file defines a Kubernetes Service for PostgreSQL.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "my-postgres.fullname" . }}
  namespace: {{ .Values.namespace }}
spec:
  selector:
    app: {{ include "my-postgres.name" . }}
  ports:
    - port: 5432
      targetPort: 5432
```

- `{{ ... }}` is Helm's Go-template syntax.
- `.Values.namespace` reads the `namespace` value from `values.yaml`.
- `include "my-postgres.fullname" .` uses a named helper template, usually defined in `templates/_helpers.tpl`.
- `selector.app` must match the corresponding labels on the PostgreSQL Pod created by the Deployment.
- Port `5432` is PostgreSQL's default port.

Make sure the helper names used in the Service match the helpers defined in your chart, and that the Deployment uses matching Pod labels.

## Configure the chart

Open `values.yaml` and review the values used by your templates. For example, if the chart references `namespace`, that key should be defined:

```yaml
namespace: default
```

The Deployment template may also read values for the container image, replicas, ports, environment variables, and storage. Define and customize the values that the templates actually reference.

Avoid committing real database passwords to Git. Use Kubernetes Secrets or an appropriate secrets-management solution for credentials.

## Check the chart

Run these commands from the directory that contains `Chart.yaml`:

```bash
helm lint .
helm template my-postgres .
```

- `helm lint .` checks the chart for common errors.
- `helm template my-postgres .` renders the templates locally so you can inspect the resulting Kubernetes YAML without installing it.

## Install the chart

First confirm that your Kubernetes cluster is available:

```bash
kubectl get nodes
```

Install the chart:

```bash
helm install my-postgres .
```

Check the release and Kubernetes resources:

```bash
helm list
kubectl get deployments
kubectl get services
kubectl get pods
```

If the chart creates resources in a specific namespace, inspect that namespace with `kubectl get all -n <namespace>`.

## Update the chart

After changing `values.yaml` or a template, apply the changes with:

```bash
helm upgrade my-postgres .
```

To preview the rendered resources before upgrading:

```bash
helm template my-postgres .
```

## Uninstall the release

```bash
helm uninstall my-postgres
```

This removes the Kubernetes resources managed by that Helm release. Persistent data may have separate lifecycle behavior, so check any PersistentVolumeClaims and storage resources before deleting database workloads.

## Important checks

1. Use the standard filenames `Chart.yaml` and `values.yaml`.
2. Ensure `Chart.yaml` has valid chart metadata, including `apiVersion`, `name`, and `version`.
3. Ensure `templates/_helpers.tpl` defines `my-postgres.name` and `my-postgres.fullname` if your templates call those helpers.
4. Ensure the Deployment's Pod labels match the Service selector.
5. Configure persistent storage and credentials appropriately for your use case.
6. Test changes with `helm lint .` and `helm template my-postgres .` before installing or upgrading.
