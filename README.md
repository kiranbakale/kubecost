# Kubecost

Kubecost is a cost monitoring and optimization platform designed for Kubernetes environments. It provides real-time insights into cluster costs, resource utilization, and efficiency. By integrating seamlessly with Kubernetes, Kubecost helps allocate costs, reduce resource waste, and optimize resource usage across workloads, namespaces, or teams. It supports hybrid and multi-cloud environments, enabling effective cost management across diverse infrastructures.

---

## Key Features

- **Cost Allocation**: Detailed tracking of resource costs by namespace, service, deployment, or pod.
- **Multi-Cloud and Hybrid Support**: Visibility and cost management for Kubernetes workloads across various cloud providers and on-premises setups.
- **Real-Time Insights**: Live cost and resource metrics for quick decision-making.
- **Resource Efficiency Recommendations**: Actionable suggestions to optimize underutilized or over-provisioned resources.
- **Custom Cost Settings**: Support for unique discounts, licenses, or infrastructure costs.
- **Alerting and Notifications**: Alerts for cost anomalies, overspending, or efficiency thresholds.
- **API and Integrations**: Seamless integration with tools like Prometheus, Grafana, and cloud cost tools.
- **Historical Reporting**: Insights into trends and budget adherence over time.

---

## Prerequisites

- **Helm Client** (version 3.1+): [Install Here](https://helm.sh/docs/intro/install/)
- **Kubectl**: [Install Here](https://kubernetes.io/docs/tasks/tools/)
- A supported Kubernetes cluster (EKS version 1.23+ requires the Amazon EBS CSI driver).
- Note: For EKS version 1.30 or later, a default StorageClass is no longer assigned.

---

## Why Kubecost?

| Feature                     | OpenCost                          | Kubecost Free                    | Kubecost Enterprise                   |
|-----------------------------|-----------------------------------|----------------------------------|---------------------------------------|
| **Description**             | Open-source cost monitoring       | Includes OpenCost features with better scaling and savings options | Full-featured with advanced integrations and support |
| **Best For**                | Small clusters                   | Medium clusters                 | Large-scale or complex infrastructure |
| **Clusters Supported**      | Unlimited (no unified view)      | Unlimited (no unified view)     | Unified multi-cluster view            |
| **Deployment**              | Pod-based                        | Helm-based                      | Helm-based                            |
| **Metric Retention**        | Limited by Prometheus environment | 15 days                        | Unlimited (supports object storage)  |
| **Support**                 | Community-driven                 | Ticket-based                    | Enterprise-level support              |

---

## How Kubecost Works

The Kubecost Helm chart deploys major components such as:
- **Kubecost Cost-Analyzer Pod**:
  - Frontend: Manages routing and UI.
  - Cost-Model: Performs cost allocation calculations.
- **Prometheus**:
  - Time-series data store for cost and health metrics.
  - Includes kube-state-metrics, node-exporter, and Alertmanager.
- **Grafana**:
  - Dashboards for cost and efficiency monitoring.

Kubecost retrieves AWS pricing data, integrates it with Prometheus metrics, and provides actionable cost insights via its dashboard.

---

## Steps to Perform POC on AWS EKS

1. **Create an EKS Cluster**:
   ```bash
   eksctl create cluster --name my-cluster --version 1.30 --region us-east-1 --nodegroup-name my-nodes --node-type t3.medium --nodes 1 --nodes-min 1 --nodes-max 1 --managed
   ```

2. **Update kubeconfig**:
   ```bash
   aws eks update-kubeconfig --region us-east-1 --name my-cluster
   ```

3. **Install AWS EBS CSI Driver**:
   a. Create an IAM service account:
   ```bash
   eksctl create iamserviceaccount \
       --name ebs-csi-controller-sa \
       --namespace kube-system \
       --cluster my-cluster \
       --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
       --approve \
       --role-only \
       --role-name AmazonEKS_EBS_CSI_DriverRole
   ```
   b. Install the add-on:
   ```bash
   eksctl create addon --name aws-ebs-csi-driver --cluster my-cluster \
       --service-account-role-arn $(aws iam get-role --role-name AmazonEKS_EBS_CSI_DriverRole --output json | jq -r '.Role.Arn') --force
   ```

4. **Install Kubecost**:
   ```bash
   helm install kubecost cost-analyzer --repo https://kubecost.github.io/cost-analyzer/ \
       --namespace kubecost --create-namespace \
       --set kubecostToken="a2lyYW5iYWthbGU5QGdtYWlsLmNvbQ==xm343yadf98"
   ```

5. **Access the Dashboard**:
   Enable port-forwarding to expose the dashboard:
   ```bash
   kubectl port-forward --namespace kubecost deployment/cost-analyzer 9090
   ```

---

## Clean-Up

- **Uninstall Kubecost**:
  ```bash
  helm uninstall kubecost -n kubecost
  ```

- **Delete the EKS Cluster**:
  ```bash
  eksctl delete cluster --name my-cluster --region us-east-1
  ```

---

## Additional Resources

- [Official Documentation](https://www.kubecost.com/install.html#show-instructions)
- [AWS Blog Post](https://aws.amazon.com/blogs/)
- [Medium Article on Kubecost](https://medium.com/)
