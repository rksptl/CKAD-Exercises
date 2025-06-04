# Helm Package Manager

This section covers Helm, the package manager for Kubernetes that helps you manage Kubernetes applications.

## Key Resources

- [Helm Documentation](https://helm.sh/docs/)
- [Helm Hub](https://hub.helm.sh/)
- [Helm Charts](https://helm.sh/docs/topics/charts/)

## Introduction to Helm

Helm is a package manager for Kubernetes that allows you to define, install, and upgrade complex Kubernetes applications. It uses charts, which are packages of pre-configured Kubernetes resources.

## Key Concepts

- **Chart**: A Helm package containing all resource definitions necessary to run an application in Kubernetes
- **Release**: An instance of a chart running in a Kubernetes cluster
- **Repository**: A place where charts can be stored and shared
- **Values**: Configuration that can be supplied to customize a chart during installation
- **Templates**: Resource definition files that use Go templating syntax to generate Kubernetes manifests

## Helm

### Exercise 1: Installing Helm

**Goal**: Learn how to install Helm and verify the installation.

**Task**: Install Helm on your local machine and verify that it's working correctly.

**Why this matters**: Installing Helm is the first step to using this powerful package manager for Kubernetes. Helm simplifies the deployment and management of applications on Kubernetes by packaging complex application components into a single unit that can be versioned, shared, and easily deployed.

<details><summary>show solution</summary>
<p>

**Step 1: Install Helm**

For Linux:

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

For macOS (using Homebrew):

```bash
brew install helm
```

For Windows (using Chocolatey):

```bash
choco install kubernetes-helm
```

**Step 2: Verify the installation**

```bash
helm version
```

You should see output similar to:

```
version.BuildInfo{Version:"v3.12.0", GitCommit:"c9f554d75773799f72ceef38c51210f1842a1dea", GitTreeState:"clean", GoVersion:"go1.20.4"}
```

**Step 3: Add a Helm repository**

```bash
helm repo add stable https://charts.helm.sh/stable
```

**Step 4: Update the repository**

```bash
helm repo update
```

**What this does**:

- Downloads and installs the Helm client
- Verifies that Helm is installed correctly
- Adds the stable repository to your Helm installation
- Updates the repository to get the latest charts
- This prepares your environment for using Helm to install and manage applications

</p>
</details>

### Exercise 2: Exploring Helm Charts

**Goal**: Learn how to search for and examine Helm charts.

**Task**: Search for available charts and inspect their details.

**Why this matters**: Before installing applications with Helm, you need to know how to find and evaluate available charts. Understanding how to search for charts and examine their structure helps you make informed decisions about which charts to use and how to configure them for your needs.

<details><summary>show solution</summary>
<p>

**Step 1: Search for charts**

```bash
helm search repo stable
```

This will list all charts in the stable repository.

**Step 2: Search for a specific chart**

```bash
helm search repo nginx
```

This will search for charts related to nginx.

**Step 3: Get information about a chart**

```bash
helm show chart stable/nginx-ingress
```

This will display the chart's metadata.

**Step 4: Show the values that can be configured**

```bash
helm show values stable/nginx-ingress
```

This will display the default values for the chart.

**Step 5: Show the README of a chart**

```bash
helm show readme stable/nginx-ingress
```

This will display the chart's README file, which typically contains usage instructions.

**What this does**:

- Searches for available charts in repositories
- Shows detailed information about specific charts
- Displays the configurable values for a chart
- Shows the documentation for a chart
- This helps you understand what charts are available and how they can be configured

</p>
</details>

### Exercise 3: Installing a Chart

**Goal**: Learn how to install a Helm chart.

**Task**: Install a chart from a repository.

**Why this matters**: Installing charts is the primary use case for Helm. Understanding how to install charts with custom configurations allows you to deploy applications to your Kubernetes cluster quickly and consistently, with the specific settings required for your environment.

<details><summary>show solution</summary>
<p>

**Step 1: Install a chart with default values**

```bash
helm install my-nginx stable/nginx-ingress
```

**Step 2: Check the status of the release**

```bash
helm status my-nginx
```

**Step 3: List all releases**

```bash
helm list
```

**Step 4: Install a chart with custom values**

Create a file named `values.yaml` with the following content:

```yaml
controller:
  replicaCount: 2
  service:
    type: NodePort
```

Then install the chart with these values:

```bash
helm install custom-nginx stable/nginx-ingress -f values.yaml
```

**Step 5: Install a chart with values specified on the command line**

```bash
helm install cli-nginx stable/nginx-ingress --set controller.replicaCount=3
```

**What this does**:

- Installs a chart with default values
- Checks the status of a release
- Lists all releases
- Installs a chart with custom values from a file
- Installs a chart with values specified on the command line
- This demonstrates different ways to install and configure Helm charts

</p>
</details>

### Exercise 4: Upgrading and Rolling Back

**Goal**: Learn how to upgrade a release and roll back if necessary.

**Task**: Upgrade a release to a new version or configuration, and then roll back if needed.

**Why this matters**: Applications evolve over time, requiring updates to their deployment configurations. Helm's upgrade and rollback capabilities allow you to safely update applications and revert changes if problems occur, minimizing downtime and ensuring reliable application management.

<details><summary>show solution</summary>
<p>

**Step 1: Upgrade a release**

```bash
helm upgrade my-nginx stable/nginx-ingress --set controller.replicaCount=3
```

**Step 2: Check the history of a release**

```bash
helm history my-nginx
```

**Step 3: Roll back to a previous revision**

```bash
helm rollback my-nginx 1
```

This will roll back to revision 1.

**Step 4: Verify the rollback**

```bash
helm status my-nginx
```

**Step 5: Upgrade and install if not exists**

```bash
helm upgrade --install my-app stable/nginx-ingress
```

This will install the chart if it doesn't exist, or upgrade it if it does.

**What this does**:

- Upgrades a release to a new configuration
- Shows the history of a release
- Rolls back to a previous revision
- Verifies the rollback was successful
- Demonstrates the upgrade-or-install pattern
- This shows how to safely update applications and recover from problematic upgrades

</p>
</details>

### Exercise 5: Uninstalling a Release

**Goal**: Learn how to uninstall a Helm release.

**Task**: Uninstall a release and verify that it's removed.

**Why this matters**: Proper cleanup of unused applications is important for maintaining a healthy Kubernetes cluster. Understanding how to safely uninstall Helm releases ensures that all associated resources are properly removed, preventing resource leaks and configuration conflicts.

<details><summary>show solution</summary>
<p>

**Step 1: Uninstall a release**

```bash
helm uninstall my-nginx
```

**Step 2: Verify that the release is uninstalled**

```bash
helm list
```

The release should no longer appear in the list.

**Step 3: Uninstall with keep history**

```bash
helm uninstall custom-nginx --keep-history
```

**Step 4: Check the history of the uninstalled release**

```bash
helm history custom-nginx
```

You should still be able to see the history.

**Step 5: Completely remove a release and its history**

```bash
helm uninstall cli-nginx
```

**What this does**:

- Uninstalls a release, removing all associated resources
- Verifies that the release is no longer active
- Demonstrates how to keep the release history for future reference
- Shows how to completely remove a release and its history
- This ensures proper cleanup of Helm-managed applications

</p>
</details>

### Exercise 6: Creating a Simple Chart

**Goal**: Learn how to create a basic Helm chart.

**Task**: Create a simple chart for a web application.

**Why this matters**: Creating custom charts allows you to package your applications in a standardized way for deployment on Kubernetes. Understanding how to build charts enables you to create reusable, versioned packages for your applications that can be shared across teams and environments.

<details><summary>show solution</summary>
<p>

**Step 1: Create a new chart**

```bash
helm create mychart
```

This will create a directory structure for a new chart.

**Step 2: Explore the chart structure**

```bash
ls -la mychart/
```

You should see files and directories like:

```
Chart.yaml
values.yaml
templates/
charts/
```

**Step 3: Modify the chart**

Edit `mychart/values.yaml` to customize the default values:

```yaml
replicaCount: 2

image:
  repository: nginx
  tag: "1.21.0"
  pullPolicy: IfNotPresent

service:
  type: NodePort
  port: 80
```

**Step 4: Lint the chart**

```bash
helm lint mychart/
```

This will check the chart for issues.

**Step 5: Install the chart**

```bash
helm install mywebapp ./mychart
```

**Step 6: Verify the installation**

```bash
kubectl get all -l app.kubernetes.io/name=mychart
```

**What this does**:

- Creates a new chart with the default structure
- Explores the chart's files and directories
- Modifies the chart's values
- Validates the chart for issues
- Installs the custom chart
- Verifies that the application is deployed correctly
- This demonstrates how to create and use your own Helm charts

</p>
</details>

### Exercise 7: Working with Chart Dependencies

**Goal**: Learn how to manage dependencies between charts.

**Task**: Create a chart that depends on another chart.

**Why this matters**: Complex applications often consist of multiple components that need to be deployed together. Chart dependencies allow you to compose applications from multiple charts, ensuring that all components are deployed in the correct order with the right configurations.

<details><summary>show solution</summary>
<p>

**Step 1: Create a new chart for the parent application**

```bash
helm create parentapp
```

**Step 2: Define a dependency in Chart.yaml**

Edit `parentapp/Chart.yaml` to add a dependency:

```yaml
apiVersion: v2
name: parentapp
description: A parent application that depends on a child chart
type: application
version: 0.1.0
appVersion: "1.0.0"
dependencies:
  - name: mychart
    version: 0.1.0
    repository: "file://../mychart"
```

**Step 3: Update dependencies**

```bash
helm dependency update parentapp/
```

This will download and package the dependencies.

**Step 4: Verify the dependencies**

```bash
ls -la parentapp/charts/
```

You should see the dependency chart as a `.tgz` file.

**Step 5: Install the parent chart**

```bash
helm install myparentapp ./parentapp
```

**Step 6: Verify the installation**

```bash
helm list
kubectl get all -l app.kubernetes.io/instance=myparentapp
```

**What this does**:

- Creates a new chart for a parent application
- Defines a dependency on another chart
- Updates the dependencies to package them
- Installs the parent chart, which also installs the dependency
- Verifies that both the parent and child applications are deployed
- This demonstrates how to compose applications from multiple charts

</p>
</details>

### Exercise 8: Using Helm Templates

**Goal**: Learn how to use Helm's templating system.

**Task**: Create and modify templates to generate Kubernetes manifests.

**Why this matters**: Helm's templating system is what makes charts dynamic and reusable. Understanding how to write and modify templates allows you to create charts that can adapt to different environments and requirements, making your application deployments more flexible and maintainable.

<details><summary>show solution</summary>
<p>

**Step 1: Examine the default templates**

```bash
ls -la mychart/templates/
```

**Step 2: Create a new template**

Create a file named `mychart/templates/configmap.yaml` with the following content:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-config
  labels:
    app: {{ .Chart.Name }}
    chart: {{ .Chart.Name }}-{{ .Chart.Version }}
    release: {{ .Release.Name }}
data:
  app.properties: |
    # Application properties
    app.name={{ .Values.appName | default "myapp" }}
    app.environment={{ .Values.environment | default "development" }}
    {{- if .Values.enableFeature }}
    app.feature.enabled=true
    {{- else }}
    app.feature.enabled=false
    {{- end }}
```

**Step 3: Add values to values.yaml**

Edit `mychart/values.yaml` to add:

```yaml
appName: "awesome-app"
environment: "staging"
enableFeature: true
```

**Step 4: Render the template**

```bash
helm template mychart/
```

This will show the rendered Kubernetes manifests.

**Step 5: Install the chart with the new template**

```bash
helm install template-demo ./mychart
```

**Step 6: Verify the ConfigMap**

```bash
kubectl get configmap template-demo-config -o yaml
```

**What this does**:

- Examines the default templates in a chart
- Creates a new template using Helm's templating syntax
- Adds values that will be used in the template
- Renders the templates to see the generated manifests
- Installs the chart with the new template
- Verifies that the ConfigMap is created correctly
- This demonstrates how to use Helm's templating system to generate dynamic Kubernetes resources

</p>
</details>
