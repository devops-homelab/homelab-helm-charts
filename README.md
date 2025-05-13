# Homelab Helm Charts

This repository contains Helm charts for managing and deploying applications and infrastructure in your homelab environment. The charts are designed to be reusable, modular, and easy to customize for various use cases.

## Repository Structure

```
charts/
└── common-application-charts/
    ├── Chart.yaml
    ├── values.yaml
    └── templates/
        ├── ingress.yaml
        ├── rollout.yaml
        └── service.yaml
```

### `common-application-charts/`
This directory contains a Helm chart template for deploying common applications. It includes the following components:

- **`Chart.yaml`**: Metadata about the chart, such as its name, version, and description.
- **`values.yaml`**: Default configuration values for the chart.
- **`templates/`**: Kubernetes resource templates for ingress, rollouts, and services.

## Usage

### Prerequisites
- [Helm](https://helm.sh/) installed on your local machine.
- A Kubernetes cluster configured and accessible.

### Installing a Chart
To install a chart, navigate to the `common-application-charts/` directory and run:

```bash
helm install <release-name> .
```
Replace `<release-name>` with a name for your Helm release.

### Customizing Values
You can override the default values in `values.yaml` by providing your own `values.yaml` file:

```bash
helm install <release-name> . -f custom-values.yaml
```

### Upgrading a Release
To upgrade an existing release with new changes:

```bash
helm upgrade <release-name> .
```

### Uninstalling a Release
To uninstall a release:

```bash
helm uninstall <release-name>
```

## Continuous Integration (CI)

This repository includes two GitHub Actions workflows to ensure the quality and reliability of Helm charts:

### 1. `chart_release.yaml`
- **Purpose**: Automates the release process for Helm charts.
- **Trigger**: Runs on a `push` event to the `main` branch.
- **Job**: 
  - **Name**: `release-chart`
  - **Functionality**: Uses a reusable workflow (`helm-release.yaml`) from the `homelab-github-reusable-workflows` repository to handle chart versioning, packaging, and publishing.
  - **Secrets**: Requires a GitHub Personal Access Token (`GH_PAT`) for authentication.

### 2. `chart_test.yaml`
- **Purpose**: Tests Helm charts for quality and correctness.
- **Trigger**: Runs on `pull_request` events (e.g., opened, reopened, labeled, synchronized) targeting the `main` branch.
- **Job**:
  - **Name**: `lint-test`
  - **Functionality**: Uses a reusable workflow (`helm-test.yaml`) from the `homelab-github-reusable-workflows` repository to perform linting and testing.

## Contributing

Contributions are welcome! If you have suggestions for improvements or new features, feel free to open an issue or submit a pull request.

## License

This repository is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

---

Happy charting!