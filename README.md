# Performance degradation patterns in Kubernetes based CICD Pipelines under resource contention
This repository contains the infrastructure-as-code and configuration files for a Master's dissertation project. The project focuses on deploying a complete monitoring stack (Prometheus, Grafana, Jenkins) on Amazon EKS and analyzing the performance of various Kubernetes application deployment patterns under different resource constraints.

## Overview
This project automates the setup of a Kubernetes cluster on AWS EKS, deploys a CI/CD pipeline (Jenkins), a full monitoring stack (Prometheus & Grafana), and a series of test application scenarios. The goal is to collect and analyze performance metrics to evaluate the efficiency of different deployment strategies in a controlled environment.

### Architecture
The infrastructure consists of:

Amazon EKS Cluster: A managed Kubernetes control plane with worker nodes.

Jenkins: Deployed in the jenkins namespace for CI/CD automation.

Monitoring Stack: Deployed in the monitoring namespace.

Prometheus: For metrics collection and storage.

Grafana: For metrics visualization and dashboarding.

Node Exporter: For collecting node-level metrics.

Kube-State-Metrics: For collecting Kubernetes object state metrics.

Test Scenarios: A series of application pods deployed with different configurations to simulate various real-world conditions and constraints.
```
📁 Repository Structure
text
dissertation-repo/
├── Jenkinsfile                 # Pipeline definition for CI/CD in Jenkins
├── jenkins/                    # Jenkins deployment configuration
│   └── jenkins.yaml            # Kubernetes manifest for Jenkins deployment
├── prometheus-grafana/         # Monitoring stack configurations
│   ├── prometheus.yaml         # Prometheus server deployment & service
│   ├── grafana.yaml            # Grafana deployment & service
│   ├── node-exporter.yaml      # DaemonSet for node-exporter
│   └── kube-state-metrics.yaml # Deployment for kube-state-metrics
└── scenario/                   # Application deployment scenarios
    ├── configmap.yaml          # Configuration data for scenarios
    ├── scenario-a.yaml         # Baseline deployment
    ├── scenario-b.yaml         # Deployment with resource requests
    ├── scenario-c-no-init.yaml # App without an init container
    ├── scenario-c-with-init.yaml # App with an init container
    ├── scenario-d-no-sidecar.yaml # App without a sidecar
    ├── scenario-d-with-sidecar.yaml # App with a sidecar (e.g., logging)
    ├── scenario-e-high.yaml    # App with high resource limits
    └── scenario-e-low.yaml     # App with low resource limits
```
### Implementation / Installation
Prerequisites
AWS CLI configured with appropriate credentials

kubectl installed

eksctl installed

An existing EKS cluster (or use eksctl to create one)

Step-by-Step Setup
Configure Kubernetes Context

```
aws eks update-kubeconfig --name <your-cluster-name> --region <your-region>
```
```
kubectl get nodes # Verify connection
```
Set up OIDC Provider for IAM Integration
```
eksctl utils associate-iam-oidc-provider --region=<your-region> --cluster=<your-cluster-name> --approve
```
Create an IAM Role for the EBS CSI Driver (for dynamic volume provisioning)
```
eksctl create iamserviceaccount \
  --name ebs-csi-controller-sa \
  --namespace kube-system \
  --cluster <your-cluster-name> \
  --role-name AmazonEKS_EBS_CSI_DriverRole \
  --role-only \
  --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
  --approve
```
Attach the EBS CSI Driver as an Amazon EKS add-on via the AWS Console or CLI.

Create Namespaces
```
kubectl create namespace jenkins
kubectl create namespace monitoring
```
Deploy Jenkins
```
kubectl apply -f jenkins/jenkins.yaml -n jenkins
```
Deploy the Monitoring Stack
```
kubectl apply -f prometheus-grafana/ -n monitoring
```
Deploy Test Scenarios (Optional, can be automated via Jenkins)
```
kubectl apply -f scenario/ -n monitoring
```
### Validation & Access
After deployment, retrieve the external endpoints to access the services:

Jenkins Dashboard:
```
kubectl get svc -n jenkins
# Access the EXTERNAL-IP for 'jenkins' service on port 8080
```
Grafana Dashboards:
```
kubectl get svc -n monitoring
# Access the EXTERNAL-IP for 'grafana-service' on port 3000
# Default login is admin/admin. You will need to configure Prometheus as a data source.
```
Prometheus Server:
```
kubectl get svc -n monitoring
# Access the EXTERNAL-IP for 'prometheus-service' on port 9090
```
Verify Scenario Pods:
```
kubectl get pods -n monitoring
# Should show all scenario pods (scenario-a, scenario-b, etc.) in Running state
```
### Evolution / CI-CD
The Jenkinsfile defines a pipeline that can automate the deployment process. The typical pipeline stages would include:

Checkout: Pull the latest code from this repository.

Deploy Scenarios: Apply the YAML files in the scenario/ directory to the monitoring namespace.

Test & Validate: Run checks to ensure all pods are healthy and services are accessible.

This automation ensures a consistent and reproducible environment for running experiments and collecting performance data.

### Analysis
With the stack deployed, performance data (CPU, memory, network I/O, latency) from each scenario pod (scenario-e-high, scenario-e-low, etc.) is collected by Prometheus via the Node Exporter and kube-state-metrics. This data can be visualized and analyzed in Grafana to draw conclusions about the performance implications of each deployment strategy and resource configuration.


**Note**: This setup is for academic and testing purposes. Please ensure AWS resources are terminated after use to avoid unnecessary costs.
