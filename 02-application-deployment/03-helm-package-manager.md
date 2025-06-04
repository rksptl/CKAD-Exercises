# Helm Package Manager

Helm is the package manager for Kubernetes, similar to what apt is for Ubuntu or npm is for Node.js. It simplifies the deployment and management of applications on Kubernetes by using charts - packages of pre-configured Kubernetes resources. This section covers how to use Helm to install, upgrade, and manage applications on Kubernetes clusters.

## Key Concepts

- **Charts**: Packages of pre-configured Kubernetes resources
- **Releases**: Instances of charts running in a cluster
- **Repositories**: Places where charts are stored and shared

## Key Resources

- [Helm Documentation](https://helm.sh/docs/)
- [Helm Hub](https://artifacthub.io/)

## Helm Basics

### Exercise 1: Creating a basic Helm chart

**Goal**: Learn how to create a new Helm chart structure.

**Task**: Create a basic Helm chart template that can be customized for your application.

**Why this matters**: Creating Helm charts is a fundamental skill for packaging Kubernetes applications. Custom charts allow you to define, version, and share application deployments using Kubernetes best practices, making application deployment repeatable and consistent across environments.

<details><summary>show solution</summary>
<p>

```bash
# Create a new Helm chart with the default template structure
helm create chart-test
```

**What this does**:

- Creates a new directory called `chart-test` with the standard Helm chart structure
- Includes template files, default values, and chart metadata
- The directory structure will include:
  - `Chart.yaml`: Contains metadata about the chart
  - `values.yaml`: Default configuration values
  - `templates/`: Directory containing template files
  - `charts/`: Directory for dependent charts

**Examining the chart structure**:

```bash
ls -la chart-test/
```

</p>
</details>

### Exercise 2: Installing a Helm chart

**Goal**: Learn how to install a Helm chart into your Kubernetes cluster.

**Task**: Install the chart created in Exercise 1.

**Why this matters**: Installing charts is the primary way to deploy applications with Helm. This skill is essential for deploying both your own applications and third-party applications available as Helm charts, enabling consistent and repeatable deployments across environments.

<details><summary>show solution</summary>
<p>

```bash
# Install the chart with a release name
helm install my-release ./chart-test
```

**What this does**:

- Installs the chart with the release name `my-release`
- Creates all the Kubernetes resources defined in the chart
- The release name is used to track this specific installation

**Verify the installation**:

```bash
helm list
```

**Check the deployed resources**:

```bash
kubectl get all -l app.kubernetes.io/instance=my-release
```

</p>
</details>

### Exercise 3: Finding pending deployments

**Goal**: Learn how to check the status of Helm releases and find pending deployments.

**Task**: Check the status of the release installed in Exercise 2 and identify any pending deployments.

**Why this matters**: Monitoring the status of Helm releases is crucial for ensuring successful deployments. Identifying pending or failed deployments quickly allows you to troubleshoot issues before they impact users, which is essential for maintaining reliable services.

<details><summary>show solution</summary>
<p>

```bash
# Check the status of the release
helm status my-release
```

**What this does**:

- Shows detailed information about the release, including:
  - Deployment status
  - Resources created
  - Notes from the chart

**Find pending deployments**:

```bash
kubectl get deployments -l app.kubernetes.io/instance=my-release
```

**Check Pods for issues**:

```bash
kubectl get pods -l app.kubernetes.io/instance=my-release
```

**For more detailed information about a specific Pod**:

```bash
kubectl describe pod <pod-name>
```

</p>
</details>

### Exercise 4: Uninstalling a release

**Goal**: Learn how to uninstall a Helm release.

**Task**: Uninstall the release created in Exercise 2.

**Why this matters**: Properly uninstalling releases is important for cleaning up resources and avoiding orphaned components. This skill ensures you can cleanly remove applications when they're no longer needed, preventing resource leaks and configuration drift.

<details><summary>show solution</summary>
<p>

```bash
# Uninstall the release
helm uninstall my-release
```

**What this does**:

- Removes all Kubernetes resources associated with the release
- Removes the release from the Helm history

**Verify the release has been uninstalled**:

```bash
helm list
kubectl get all -l app.kubernetes.io/instance=my-release
```

</p>
</details>

### Exercise 5: Upgrading a chart

**Goal**: Learn how to upgrade a Helm release to a new version or with different values.

**Task**: Install a chart and then upgrade it with modified values.

**Why this matters**: Upgrading applications is a common operational task. Helm's upgrade mechanism provides a controlled way to update applications with new versions or configurations while maintaining release history, enabling easy rollbacks if issues occur.

<details><summary>show solution</summary>
<p>

**Step 1: Install a chart**

```bash
helm install upgrade-test ./chart-test
```

**Step 2: Create a custom values file**

Create a file called `new-values.yaml` with the following content:

```yaml
replicaCount: 2
service:
  type: NodePort
```

**Step 3: Upgrade the release with the new values**

```bash
helm upgrade upgrade-test ./chart-test -f new-values.yaml
```

**What this does**:

- Updates the release with new configuration values
- Changes the replica count to 2
- Changes the service type to NodePort

**Verify the upgrade**:

```bash
helm list
kubectl get deployments -l app.kubernetes.io/instance=upgrade-test
kubectl get services -l app.kubernetes.io/instance=upgrade-test
```

</p>
</details>

### Exercise 6: Managing repositories

**Goal**: Learn how to manage Helm chart repositories.

**Task**: Add, list, and remove Helm repositories.

**Why this matters**: Helm repositories are where charts are stored and shared. Managing repositories is essential for accessing both official and custom charts, allowing you to leverage the broader Kubernetes ecosystem and share charts within your organization.

<details><summary>show solution</summary>
<p>

**Step 1: Add a repository**

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

**Step 2: List repositories**

```bash
helm repo list
```

**Step 3: Update repositories**

```bash
helm repo update
```

**Step 4: Remove a repository**

```bash
helm repo remove bitnami
```

**What this does**:

- `helm repo add` - Adds a new chart repository with a name and URL
- `helm repo list` - Lists all configured repositories
- `helm repo update` - Updates the local cache of charts from all repositories
- `helm repo remove` - Removes a repository from the configuration

</p>
</details>

### Exercise 7: Downloading a chart

**Goal**: Learn how to download a chart from a repository without installing it.

**Task**: Add a repository and download a chart from it.

**Why this matters**: Downloading charts allows you to inspect and modify them before deployment. This is particularly useful for security reviews, customization, or when you need to adapt a chart to your specific environment requirements.

<details><summary>show solution</summary>
<p>

**Step 1: Add the Bitnami repository**

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

**Step 2: Update the repositories**

```bash
helm repo update
```

**Step 3: Pull the chart**

```bash
helm pull bitnami/nginx
```

**What this does**:

- Downloads the chart as a `.tgz` file in the current directory
- Allows you to extract and examine the chart contents before installation

**To extract and examine the chart**:

```bash
tar -xzf nginx-*.tgz
ls -la nginx/
```

</p>
</details>

### Exercise 8: Adding a repository with authentication

**Goal**: Learn how to add a private Helm repository that requires authentication.

**Task**: Add a repository that requires a username and password.

**Why this matters**: Many organizations use private repositories to store proprietary charts. Understanding how to configure authentication is essential for accessing these repositories and maintaining security while using private charts.

<details><summary>show solution</summary>
<p>

```bash
helm repo add private-repo https://charts.example.com --username=myuser --password=mypassword
```

**What this does**:

- Adds a repository with authentication credentials
- Stores the credentials securely in the Helm configuration

**Alternative using environment variables**:

```bash
export HELM_REPO_USERNAME=myuser
export HELM_REPO_PASSWORD=mypassword
helm repo add private-repo https://charts.example.com
```

> **Note**: Be careful with credentials in scripts or command history. Consider using a credential manager or Kubernetes secrets for production environments.

</p>
</details>

### Exercise 9: Viewing chart configuration

**Goal**: Learn how to view the default values and structure of a chart.

**Task**: Examine the configuration options of a chart without installing it.

**Why this matters**: Understanding a chart's configuration options is crucial before deployment. This skill allows you to properly customize applications for your environment and ensure they meet your requirements without trial-and-error deployments.

<details><summary>show solution</summary>
<p>

**Step 1: Add a repository if not already added**

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

**Step 2: View the chart's default values**

```bash
helm show values bitnami/nginx
```

**Step 3: View the chart's README**

```bash
helm show readme bitnami/nginx
```

**Step 4: View all information about the chart**

```bash
helm show all bitnami/nginx
```

**What this does**:

- `helm show values` - Displays the default configuration values for the chart
- `helm show readme` - Shows the chart's README file with documentation
- `helm show all` - Shows all information about the chart, including values, README, and chart metadata

</p>
</details>

### Exercise 10: Installing a chart with custom parameters

**Goal**: Learn how to install a chart with custom configuration values.

**Task**: Install a chart with custom values provided on the command line.

**Why this matters**: Most applications require customization for specific environments. Being able to override default values during installation allows you to adapt charts to your needs without modifying the chart itself, maintaining compatibility with future updates.

<details><summary>show solution</summary>
<p>

**Step 1: Install a chart with custom values**

```bash
helm install custom-nginx bitnami/nginx \
  --set replicaCount=2 \
  --set service.type=NodePort
```

**Step 2: Verify the installation**

```bash
kubectl get deployments,services -l app.kubernetes.io/instance=custom-nginx
```

**What this does**:

- Installs the nginx chart with the release name `custom-nginx`
- Sets the replica count to 2 instead of the default
- Changes the service type to NodePort
- The `--set` flag allows you to override specific values without creating a values file

**Alternative using a values file**:

Create a file called `custom-values.yaml`:

```yaml
replicaCount: 2
service:
  type: NodePort
```

Then install with:

```bash
helm install custom-nginx bitnami/nginx -f custom-values.yaml
```

</p>
</details>
