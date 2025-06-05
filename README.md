[![Software License](https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square)](LICENSE)

# CKAD Exercises

A comprehensive set of exercises to help prepare for the [Certified Kubernetes Application Developer](https://www.cncf.io/certification/ckad/) exam, offered by the Cloud Native Computing Foundation. This repository has been completely reorganized to align with the official CKAD exam domains and competencies.

These exercises serve as practical learning resources for mastering Kubernetes application development concepts. Each section includes clear goals, tasks, and explanations of why each concept matters in real-world scenarios.

## Key Features

- **Docker and containerd Focus**: All container examples use Docker and containerd (with nerdctl) as the container runtimes
- **Imperative Commands**: Where possible, exercises include both imperative command options and declarative manifest-based approaches
- **Meaningful Resource Names**: All examples use descriptive, meaningful names for resources to improve clarity
- **Exam-Aligned Content**: Exercises closely match the tasks you'll encounter in the actual CKAD exam

## About the CKAD Exam

- **Duration**: 2 hours
- **Format**: Performance-based (hands-on tasks in a live Kubernetes environment)
- **Passing Score**: 66%
- **Environment**: Remote proctored with access to official Kubernetes documentation only

During the exam, you are only allowed to refer to official documentation from a browser window within the exam VM. Each exercise section includes helpful links to relevant Kubernetes documentation.

## Contents (Aligned with CKAD Exam Domains)

### 1. Application Design and Build (20%)

- [Container Images](01-application-design/01-container-images.md)
- [Workload Resources](01-application-design/02-workload-resources.md)
- [Multi-Container Pods](01-application-design/03-multi-container-pods.md)
- [Volumes](01-application-design/04-volumes.md)

### 2. Application Deployment (20%)

- [Deployment Strategies](02-application-deployment/01-deployment-strategies.md)
- [Rolling Updates and Rollbacks](02-application-deployment/02-rolling-updates.md)
- [Helm Package Manager](02-application-deployment/03-helm-package-manager.md)

### 3. Application Observability and Maintenance (15%)

- [Probes](03-application-observability/01-probes.md)
- [Logging and Monitoring](03-application-observability/02-logging-monitoring.md)
- [Debugging](03-application-observability/03-debugging.md)

### 4. Application Environment, Configuration and Security (25%)

- [ConfigMaps and Secrets](04-environment-config-security/01-configmaps-secrets.md)
- [Configuration Management](04-environment-config-security/02-configuration-management.md)
- [Security Contexts](04-environment-config-security/03-security-contexts.md)
- [Resource Requirements](04-environment-config-security/04-resource-requirements.md)

### 5. Services and Networking (20%)

- [Services](05-services-networking/01-services.md)
- [Network Policies](05-services-networking/02-network-policies.md)
- [Ingress](05-services-networking/03-ingress.md)

### 6. State Persistence (10%)

- [Volumes](06-state-persistence/01-volumes.md)
- [Persistent Volumes](06-state-persistence/02-persistent-volumes.md)
- [Storage Classes](06-state-persistence/03-storage-classes.md)

### 7. Helm Package Manager (Additional Content)

- [Helm Basics](07-helm-package-manager/01-helm-basics.md)

## Repository Organization

This repository is organized according to the official CKAD exam domains and their respective weightings. Each domain directory contains multiple markdown files covering specific competencies within that domain.

## Exercise Format

Each exercise follows a consistent format:

1. **Goal**: What you'll learn from the exercise
2. **Task**: The specific actions to complete
3. **Why this matters**: Real-world context for the concept
4. **Solution**: Step-by-step instructions (hidden by default)

Exercises include both imperative kubectl commands and declarative YAML manifests where appropriate, with clear guidance on when to use each approach. Container examples use both Docker and containerd (nerdctl) commands for flexibility.

## Study Tips

1. **Practice hands-on**: Set up a local Kubernetes environment (Minikube, Kind, or k3s)
2. **Time yourself**: Practice completing tasks within time constraints
3. **Master imperative commands**: Learn to create resources quickly using kubectl imperative commands
4. **Know when to use YAML**: Understand which resources require manifest files and which can be created imperatively
5. **Use kubectl efficiently**: Learn shortcuts and kubectl command syntax
6. **Understand YAML**: Practice writing Kubernetes manifests without relying on examples
7. **Focus on troubleshooting**: Practice debugging common issues

## Contributing

Contributions are welcome! If you find an error, have an alternative solution, or want to add new exercises, please feel free to submit a pull request. Please follow the existing format for consistency.

If this repo has helped you in any way, feel free to post on [discussions](https://github.com/rksptl/CKAD-Exercises/discussions) or buy me a coffee!

<a href="https://www.buymeacoffee.com/rksptl" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/default-orange.png" alt="Buy Me A Coffee" height="41" width="174"></a>

## License

This repository is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

Copyright (c) 2025 Rakesh Kumar
