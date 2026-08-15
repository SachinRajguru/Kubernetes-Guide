
## Kubernetes Cluster Deployment Using KOPS on AWS

**Hands-On Lab — Provisioning, Validating, Accessing, and Managing a Kubernetes Cluster on AWS**

## Table of Contents

1. [Lab Overview](#1-lab-overview)
2. [Lab Objective](#2-lab-objective)
3. [Architecture](#3-architecture)
4. [Prerequisites](#4-prerequisites)
5. [Lab Environment](#5-lab-environment)
6. [Step 1 — Verify Required Tools](#6-step-1--verify-required-tools)
7. [Step 2 — Configure AWS CLI](#7-step-2--configure-aws-cli)
8. [Step 3 — Verify AWS Access](#8-step-3--verify-aws-access)
9. [Step 4 — Create KOPS State Store](#9-step-4--create-kops-state-store)
10. [Step 5 — Configure DNS](#10-step-5--configure-dns)
11. [Step 6 — Create Kubernetes Cluster](#11-step-6--create-kubernetes-cluster)
12. [Step 7 — Review Cluster Configuration](#12-step-7--review-cluster-configuration)
13. [Step 8 — Build the Kubernetes Cluster](#13-step-8--build-the-kubernetes-cluster)
14. [Step 9 — Validate the Cluster](#14-step-9--validate-the-cluster)
15. [Step 10 — Verify Kubernetes Nodes](#15-step-10--verify-kubernetes-nodes)
16. [Step 11 — Verify System Pods](#16-step-11--verify-system-pods)
17. [Step 12 — Access the Kubernetes Cluster](#17-step-12--access-the-kubernetes-cluster)
18. [Step 13 — Basic KOPS Lifecycle Operations](#18-step-13--basic-kops-lifecycle-operations)
19. [Cost and Cleanup](#19-cost-and-cleanup)
20. [Troubleshooting](#20-troubleshooting)
21. [Expected Outcome](#21-expected-outcome)
22. [Lab Completion Checklist](#22-lab-completion-checklist)

## 1. Lab Overview

In this hands-on lab, we will provision a **Kubernetes cluster on AWS using KOPS**.

> Unlike local Kubernetes tools such as Minikube or Kind, KOPS provisions Kubernetes infrastructure on AWS.

During this lab, we will work with:

* AWS EC2
* Amazon S3
* Amazon Route 53
* IAM
* KOPS
* kubectl
* Kubernetes Control Plane
* Kubernetes Worker Nodes

The objective is not only to create the cluster, but also to understand the basic **KOPS cluster lifecycle** used in real-world environments.

## 2. Lab Objective

By completing this lab, we will learn how to:

* Configure AWS access.
* Create an S3 bucket for KOPS state storage.
* Configure DNS for the Kubernetes cluster.
* Create a Kubernetes cluster using KOPS.
* Review the generated cluster configuration.
* Provision AWS infrastructure.
* Validate the Kubernetes cluster.
* Verify Worker Nodes.
* Verify Kubernetes system Pods.
* Access the cluster using kubectl.
* Perform basic KOPS lifecycle operations.
* Safely delete the cluster and clean up AWS resources.

## 3. Architecture

The high-level architecture for this lab is:

```text
                                  AWS Cloud
                                     |
                                     |
                                 KOPS CLI
                                     |
                                     |
                                  AWS API
                                     |
                                     |
                      +--------------+--------------+
                      |                             |
                      |                             |
                S3 State Store                   Route 53
                  KOPS State                       DNS
                                                    |
                                                    |
                                          Kubernetes Cluster
                                                    |
                                                    |
                                      +-------------+-------------+
                                      |                           |
                                      |                           |
                              Control Plane                Worker Nodes
                              Kubernetes                   Application
                              Components                       Pods
```

> The important point is that **KOPS uses AWS APIs to provision and manage the Kubernetes infrastructure**, while S3 stores the KOPS cluster state.

Route 53 provides DNS support when we use a domain or hosted zone for the Kubernetes cluster.

## 4. Prerequisites

> Before starting this lab, ensure that the required environment has already been prepared.

The following tools should be available:

| Tool     | Purpose                                  |
| -------- | ---------------------------------------- |
| Python 3 | Required dependency for AWS tooling      |
| AWS CLI  | Communicate with AWS                     |
| kubectl  | Manage the Kubernetes cluster            |
| KOPS     | Create and manage the Kubernetes cluster |

The setup instructions are maintained separately in:

```text
setup/
```

The setup document covers:

* Required packages
* Python
* AWS CLI
* kubectl
* KOPS
* IAM permissions
* AWS CLI configuration

## 5. Lab Environment

We can run the lab from:

* An Ubuntu EC2 instance
* A Linux workstation
* macOS
* Windows using WSL

For this lab, we will use an AWS environment.

Example:

```text
AWS Account
     |
     |
Ubuntu EC2 Instance
     |
     +---- AWS CLI
     |
     +---- kubectl
     |
     +---- KOPS
     |
     |
AWS Infrastructure
     |
     |
Kubernetes Cluster
```

## 6. Step 1 — Verify Required Tools

> Before starting the cluster deployment, verify that the required tools are installed.

### Verify Python

```bash
python3 --version
```

Expected:

```text
Python 3.x.x
```

### Verify AWS CLI

```bash
aws --version
```

### Verify kubectl

```bash
kubectl version --client
```

### Verify KOPS

```bash
kops version
```

> If any tool is missing, complete the corresponding setup instructions before continuing.

## 7. Step 2 — Configure AWS CLI

Configure AWS credentials:

```bash
aws configure
```

Provide:

```text
AWS Access Key ID:
AWS Secret Access Key:
Default Region:
Output Format:
```

For this lab, we will use:

```text
us-east-1
```

> **Security Note:** Do not commit AWS access keys or secret keys to Git repositories.

## 8. Step 3 — Verify AWS Access

Verify that the configured AWS credentials can access the AWS account:

```bash
aws sts get-caller-identity
```

Example response:

```json
{
    "Account": "xxxxxxxxxxxx",
    "UserId": "xxxxxxxx",
    "Arn": "arn:aws:iam::xxxxxxxxxxxx:user/demo"
}
```

The command should return the AWS account and identity associated with the configured credentials.

## 9. Step 4 — Create KOPS State Store

> KOPS requires a state store to maintain information about the Kubernetes cluster.

For AWS, we will use **Amazon S3**.

Create an S3 bucket with a unique name:

```bash
aws s3api create-bucket \
  --bucket <unique-kops-state-store-name> \
  --region us-east-1
```

Replace:

```text
<unique-kops-state-store-name>
```

with a globally unique S3 bucket name.

For example:

```text
kops-demo-cluster-state-store
```

Verify the bucket:

```bash
aws s3 ls
```

We will reference the bucket later as:

```text
s3://<unique-kops-state-store-name>
```

Example:

```text
s3://kops-demo-cluster-state-store
```

## 10. Step 5 — Configure DNS

> KOPS uses DNS for Kubernetes cluster discovery and cluster management.

For learning or non-production environments, we can use:

```text
k8s.local
```

For a production-style environment, we should use a domain or subdomain managed through **Amazon Route 53**.

Example:

```text
k8s.example.com
```

or:

```text
kubernetes.example.com
```

### Create a Route 53 Hosted Zone

If the domain is managed through Route 53, create a hosted zone for the required domain.

Example:

```bash
aws route53 create-hosted-zone \
  --name example.com \
  --caller-reference kops-$(date +%s)
```

Verify the hosted zone:

```bash
aws route53 list-hosted-zones
```

The domain can then be used as the DNS zone for the KOPS cluster.

> **Note:** The exact DNS configuration depends on whether the domain is already registered and managed in Route 53.

For a learning environment, we can continue using:

```text
k8s.local
```

without configuring a public production domain.

## 11. Step 6 — Create Kubernetes Cluster

Now we will create the KOPS cluster configuration.

For a learning environment, we can use:

```bash
kops create cluster \
  --name=demok8scluster.k8s.local \
  --state=s3://<unique-kops-state-store-name> \
  --zones=us-east-1a \
  --node-count=1 \
  --node-size=t2.micro \
  --master-size=t2.micro \
  --master-volume-size=8 \
  --node-volume-size=8
```

Replace:

```text
<unique-kops-state-store-name>
```

with the S3 bucket created earlier.

For example:

```bash
--state=s3://kops-demo-cluster-state-store
```

> **Important:**

> The instance types above are intentionally small for a learning environment.

> They are **not production recommendations**.

> Production clusters require appropriate instance sizes, multiple nodes, high availability, storage, networking, security, and capacity planning.

## 12. Step 7 — Review Cluster Configuration

Before provisioning AWS resources, review the generated configuration:

```bash
kops edit cluster demok8scluster.k8s.local
```

Review:

* Kubernetes version
* EC2 instance types
* Number of Worker Nodes
* Networking
* Availability Zones
* Cluster configuration
* DNS configuration

> This step is important because KOPS may generate additional AWS resources during cluster creation.

## 13. Step 8 — Build the Kubernetes Cluster

Once the configuration has been reviewed, provision the infrastructure:

```bash
kops update cluster \
  demok8scluster.k8s.local \
  --yes \
  --state=s3://<unique-kops-state-store-name>
```

KOPS will use the AWS APIs to create the required infrastructure.

Depending on the configuration, this can include:

* VPC resources
* EC2 instances
* Security groups
* IAM resources
* Networking
* Kubernetes Control Plane
* Worker Nodes
* Kubernetes certificates
* DNS configuration

The provisioning process may take several minutes.

## 14. Step 9 — Validate the Cluster

After the infrastructure has been provisioned, validate the Kubernetes cluster:

```bash
kops validate cluster \
  demok8scluster.k8s.local \
  --state=s3://<unique-kops-state-store-name>
```

> A successful validation indicates that the Kubernetes cluster is responding and the expected components are available.

## 15. Step 10 — Verify Kubernetes Nodes

Check the Worker Nodes:

```bash
kubectl get nodes
```

Example:

```text
NAME                                      STATUS   ROLES
ip-10-0-1-25.us-east-1.compute.internal   Ready    <none>
```

The Node should eventually reach:

```text
STATUS = Ready
```

We can also get additional information:

```bash
kubectl get nodes -o wide
```

## 16. Step 11 — Verify System Pods

Kubernetes system components run as Pods on the cluster.

Check all namespaces:

```bash
kubectl get pods -A
```

> We should see Kubernetes system workloads running across the cluster.

We can also inspect the cluster:

```bash
kubectl get all -A
```

This helps confirm that the cluster is functioning beyond simply having a `Ready` Node.

## 17. Step 12 — Access the Kubernetes Cluster

Verify the current kubectl context:

```bash
kubectl config current-context
```

Expected:

```text
demok8scluster.k8s.local
```

List available contexts:

```bash
kubectl config get-contexts
```

If required, switch to the KOPS cluster:

```bash
kubectl config use-context demok8scluster.k8s.local
```

Now verify cluster access:

```bash
kubectl cluster-info
```

We should receive information about the Kubernetes Control Plane and cluster services.

## 18. Step 13 — Basic KOPS Lifecycle Operations

KOPS is not only used to create clusters.

It also provides lifecycle management capabilities.

### List KOPS Clusters

```bash
kops get clusters \
  --state=s3://<unique-kops-state-store-name>
```

### View Cluster Configuration

```bash
kops get cluster \
  demok8scluster.k8s.local \
  --state=s3://<unique-kops-state-store-name>
```

### Edit Cluster Configuration

```bash
kops edit cluster \
  demok8scluster.k8s.local \
  --state=s3://<unique-kops-state-store-name>
```

### Update Cluster

After changing configuration:

```bash
kops update cluster \
  demok8scluster.k8s.local \
  --state=s3://<unique-kops-state-store-name>
```

If the changes should be applied:

```bash
kops update cluster \
  demok8scluster.k8s.local \
  --state=s3://<unique-kops-state-store-name> \
  --yes
```

### Validate Cluster

```bash
kops validate cluster \
  demok8scluster.k8s.local \
  --state=s3://<unique-kops-state-store-name>
```

These operations demonstrate the **cluster lifecycle management** capability of KOPS.

## 19. Cost and Cleanup

> KOPS creates AWS resources that may incur charges.

Typical resources can include:

* EC2 instances
* EBS volumes
* VPC resources
* Load Balancers
* S3 resources
* Route 53 resources

> For learning environments, clean up the resources after completing the lab.

### Delete the KOPS Cluster

```bash
kops delete cluster \
  demok8scluster.k8s.local \
  --state=s3://<unique-kops-state-store-name> \
  --yes
```

> After deletion, verify that the expected AWS resources have been removed.

### Delete the KOPS State Store

If the S3 state bucket is no longer required, remove its contents first and then delete the bucket separately.

Example:

```bash
aws s3 rm s3://<unique-kops-state-store-name> --recursive
```

Then:

```bash
aws s3api delete-bucket \
  --bucket <unique-kops-state-store-name> \
  --region us-east-1
```

If a Route 53 hosted zone was created specifically for this lab and is no longer required, clean up its DNS records and hosted zone separately.

> **Important:** Do not delete shared production resources or DNS zones that are used by other environments.

## 20. Troubleshooting

### Issue 1 — AWS AccessDenied

Check the configured identity:

```bash
aws sts get-caller-identity
```

Verify that the AWS identity has the permissions required by KOPS.

### Issue 2 — Cluster Validation Fails

Run:

```bash
kops validate cluster \
  demok8scluster.k8s.local \
  --state=s3://<unique-kops-state-store-name>
```

Check EC2 instances:

```bash
aws ec2 describe-instances
```

Also check the KOPS cluster configuration:

```bash
kops get cluster \
  demok8scluster.k8s.local \
  --state=s3://<unique-kops-state-store-name>
```

### Issue 3 — kubectl Cannot Access the Cluster

Check the current context:

```bash
kubectl config current-context
```

List contexts:

```bash
kubectl config get-contexts
```

Switch to the KOPS cluster if required:

```bash
kubectl config use-context demok8scluster.k8s.local
```

### Issue 4 — Worker Node Is Not Ready

Check:

```bash
kubectl get nodes
```

Then:

```bash
kubectl describe node <node-name>
```

Also check the KOPS validation output:

```bash
kops validate cluster \
  demok8scluster.k8s.local \
  --state=s3://<unique-kops-state-store-name>
```

## 21. Expected Outcome

At the end of this lab, we should have:

```text
AWS
 |
 +-- S3
 |    |
 |    +-- KOPS State Store
 |
 +-- Route 53
 |    |
 |    +-- DNS Configuration
 |
 +-- VPC
      |
      +-- Control Plane
      |
      +-- Worker Node(s)
             |
             +-- Kubernetes Pods
```

The cluster should be accessible using:

```bash
kubectl get nodes
```

and the Node status should be:

```text
Ready
```

We should also be able to validate the cluster using:

```bash
kops validate cluster
```

## 22. Lab Completion Checklist

Before considering the lab complete, verify the following:

* [ ] Python 3 is installed.
* [ ] AWS CLI is installed.
* [ ] kubectl is installed.
* [ ] KOPS is installed.
* [ ] AWS CLI is configured.
* [ ] AWS identity was verified.
* [ ] KOPS S3 state store was created.
* [ ] DNS configuration was prepared.
* [ ] KOPS cluster configuration was created.
* [ ] Cluster configuration was reviewed.
* [ ] AWS infrastructure was provisioned.
* [ ] KOPS cluster validation succeeded.
* [ ] Kubernetes Nodes are `Ready`.
* [ ] Kubernetes system Pods are running.
* [ ] kubectl can access the cluster.
* [ ] Basic KOPS lifecycle operations were verified.
* [ ] Lab resources were cleaned up after completion.

## Official Documentation

For additional reference:

* **KOPS Documentation** — [KOPS Official Documentation](https://kops.sigs.k8s.io/)
* **KOPS GitHub Repository** — [KOPS on GitHub](https://github.com/kubernetes/kops)
* **Kubernetes Documentation** — [Kubernetes Documentation](https://kubernetes.io/docs/)
* **kubectl Documentation** — [kubectl Documentation](https://kubernetes.io/docs/reference/kubectl/)
* **AWS CLI Documentation** — [AWS CLI Documentation](https://docs.aws.amazon.com/cli/)
* **Amazon S3 Documentation** — [Amazon S3 Documentation](https://docs.aws.amazon.com/s3/)
* **Amazon Route 53 Documentation** — [Amazon Route 53 Documentation](https://docs.aws.amazon.com/route53/)

## Lab Completion

At this point, we have provisioned a Kubernetes cluster on AWS using KOPS and verified that the cluster is operational.

> The important takeaway is that KOPS is not simply an installation utility. It provides **Kubernetes cluster lifecycle management**, including provisioning, configuration, updates, validation, and deletion.

This lab provides the practical foundation for understanding how self-managed Kubernetes environments are provisioned and operated on AWS.
