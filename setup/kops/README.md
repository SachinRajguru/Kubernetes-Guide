
## Kubernetes Installation Using KOPS on AWS

**A Practical Guide to Provisioning and Managing a Kubernetes Cluster on AWS**

## Table of Contents

1. [Introduction](#1-introduction)
2. [Architecture and Installation Flow](#2-architecture-and-installation-flow)
3. [Prerequisites](#3-prerequisites)
4. [Create an EC2 Instance](#4-create-an-ec2-instance)
5. [Install Python 3](#5-install-python-3)
6. [Install kubectl](#6-install-kubectl)
7. [Install AWS CLI](#7-install-aws-cli)
8. [Install KOPS](#8-install-kops)
9. [Configure AWS IAM Permissions](#9-configure-aws-iam-permissions)
10. [Configure AWS CLI](#10-configure-aws-cli)
11. [Create KOPS State Store](#11-create-kops-state-store)
12. [Configure DNS for Production](#12-configure-dns-for-production)
13. [Create Kubernetes Cluster](#13-create-kubernetes-cluster)
14. [Review Cluster Configuration](#14-review-cluster-configuration)
15. [Build the Kubernetes Cluster](#15-build-the-kubernetes-cluster)
16. [Validate the Kubernetes Cluster](#16-validate-the-kubernetes-cluster)
17. [Verify Kubernetes Nodes](#17-verify-kubernetes-nodes)
18. [Manage the KOPS Cluster Lifecycle](#18-manage-the-kops-cluster-lifecycle)
19. [Cost and Cleanup](#19-cost-and-cleanup)
20. [Troubleshooting](#20-troubleshooting)
21. [Production Considerations](#21-production-considerations)
22. [Official Documentation](#22-official-documentation)

- [Final Takeaways](#final-takeaways)

## 1. Introduction

KOPS stands for:

> **Kubernetes Operations**

KOPS is an open-source tool used to create, configure, upgrade, and manage Kubernetes clusters on AWS.

In this setup, we will use KOPS to provision a Kubernetes cluster on AWS infrastructure.

The overall workflow is:

```text
                 Developer / DevOps Engineer
                           |
                           |
                        KOPS CLI
                           |
                           |
                        AWS APIs
                           |
                           |
            +--------------+--------------+
            |                             |
            |                             |
      S3 State Store                AWS Resources
      Cluster State                       |
                                          |
                                  Kubernetes Cluster
                                          |
                                          |
                                +---------+---------+
                                |                   |
                                |                   |
                          Control Plane       Worker Nodes
                                                    |
                                                    |
                                             Application Pods
```

This setup demonstrates how a self-managed Kubernetes cluster can be provisioned and managed on AWS.

> **Important:** KOPS is a self-managed Kubernetes solution. Unlike Amazon EKS, AWS does not manage the Kubernetes control plane for us.

## 2. Architecture and Installation Flow

The high-level KOPS architecture is:

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
        S3 Bucket                     EC2 Instances
        State Store                  Kubernetes Nodes
                                            |
                                            |
                              +-------------+-------------+
                              |                           |
                              |                           |
                        Control Plane               Worker Nodes
                        Kubernetes                  Application
                        Components                      Pods
```

The installation process follows this sequence:

```text
Create Management EC2 Instance
            |
            |
Install Required Tools
            |
            +--> Python 3
            +--> kubectl
            +--> AWS CLI
            +--> KOPS
            |
            |
Configure IAM Permissions
            |
            |
Configure AWS CLI
            |
            |
Create S3 KOPS State Store
            |
            |
Configure DNS
            |
            |
Create KOPS Cluster Configuration
            |
            |
Review Configuration
            |
            |
Build Kubernetes Infrastructure
            |
            |
Validate Cluster
            |
            |
Verify Worker Nodes
```

## 3. Prerequisites

Before starting, we need:

### AWS Requirements

* An AWS Account
* IAM permissions to create the required AWS resources
* An AWS Region
* Access to EC2, S3, IAM, VPC, and Route 53 as required

### Management Machine

We can run KOPS from:

* An Ubuntu EC2 instance
* Linux
* macOS
* Windows using WSL

For a production-like learning environment, we will use an **Ubuntu EC2 instance as the management machine**.

The management machine is used to execute:

```text
AWS CLI
kubectl
KOPS
```

It does **not** necessarily become part of the Kubernetes cluster.

## 4. Create an EC2 Instance

We can create an Ubuntu EC2 instance that will act as our KOPS management machine.

Example:

```text
AWS Cloud
    |
    |
Management EC2 Instance
    |
    +--> Python 3
    +--> AWS CLI
    +--> kubectl
    +--> KOPS
```

Use a current supported Ubuntu LTS release.

After connecting to the instance:

```bash
ssh ubuntu@<ec2-public-ip>
```

Verify the operating system:

```bash
cat /etc/os-release
```

## 5. Install Python 3

> Python 3 is commonly available by default on modern Ubuntu releases.

Verify:

```bash
python3 --version
```

If Python 3 is not installed:

```bash
sudo apt-get update
sudo apt-get install -y python3
```

Verify again:

```bash
python3 --version
```

> **Note:** Modern AWS CLI v2 does not require Python to be installed separately. We include Python 3 because it is a common prerequisite in AWS/KOPS management environments and may be required by supporting tooling.

## 6. Install kubectl

`kubectl` is the Kubernetes command-line tool used to communicate with the Kubernetes API Server.

Install the required packages:

```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl apt-transport-https gnupg
```

Create the keyring directory:

```bash
sudo mkdir -p -m 755 /etc/apt/keyrings
```

Download the Kubernetes signing key:

```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key \
  | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

Add the Kubernetes repository:

```bash
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' \
  | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

Install `kubectl`:

```bash
sudo apt-get update
sudo apt-get install -y kubectl
```

Verify:

```bash
kubectl version --client
```

> **Note:** The Kubernetes repository version should be aligned with the Kubernetes version we intend to use. Avoid blindly using an outdated repository version for new production deployments.

## 7. Install AWS CLI

AWS CLI allows us to communicate with AWS services from the command line.

Install AWS CLI:

```bash
sudo snap install aws-cli --classic
```

Verify:

```bash
aws --version
```

Example:

```text
aws-cli/2.x.x ...
```

We will configure AWS CLI after the required IAM permissions have been provided.

## 8. Install KOPS

KOPS is the primary tool used to create and manage our Kubernetes cluster.

Download the latest KOPS Linux binary:

```bash
curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
```

Make it executable:

```bash
chmod +x kops-linux-amd64
```

Move it to the system path:

```bash
sudo mv kops-linux-amd64 /usr/local/bin/kops
```

Verify:

```bash
kops version
```

## 9. Configure AWS IAM Permissions

KOPS requires AWS permissions to create and manage infrastructure.

For a learning environment, we may use the following AWS managed policies:

```text
AmazonEC2FullAccess
AmazonS3FullAccess
IAMFullAccess
AmazonVPCFullAccess
```

If Route 53 is being used for DNS management, the required Route 53 permissions must also be available.

For example:

```text
AmazonRoute53FullAccess
```

> **Production recommendation:** Avoid using broad `FullAccess` policies for long-lived production environments. Follow the principle of least privilege and provide only the permissions required by the KOPS workflow.

The AWS identity used by KOPS must be able to create and manage the required resources.

## 10. Configure AWS CLI

Run:

```bash
aws configure
```

Provide:

```text
AWS Access Key ID:
AWS Secret Access Key:
Default region name:
Default output format:
```

Example:

```text
Default region name: us-east-1
```

Verify AWS access:

```bash
aws sts get-caller-identity
```

Expected output:

```json
{
    "Account": "xxxxxxxxxxxx",
    "UserId": "xxxxxxxx",
    "Arn": "arn:aws:iam::xxxxxxxxxxxx:user/demo"
}
```

This confirms that the management machine can authenticate with AWS.

## 11. Create KOPS State Store

KOPS requires a state store to maintain cluster configuration and state.

For AWS deployments, we use an S3 bucket.

Use a unique bucket name:

```bash
aws s3api create-bucket \
  --bucket <unique-kops-state-store-name> \
  --region <aws-region>
```

Example:

```bash
aws s3api create-bucket \
  --bucket kops-sachin-cluster-state-store \
  --region us-east-1
```

Verify:

```bash
aws s3 ls
```

The state store will contain KOPS cluster information such as:

* Cluster configuration
* Certificates
* Secrets
* Cluster state
* KOPS metadata

### Production Recommendation

For production environments, consider enabling:

* S3 versioning
* Server-side encryption
* Restricted IAM access
* Appropriate bucket policies

## 12. Configure DNS for Production

KOPS can use different DNS approaches depending on the environment.

For a learning environment, we can use:

```text
k8s.local
```

Example:

```text
demok8scluster.k8s.local
```

For an organization or production environment, we should use an organization-controlled domain.

For example:

```text
k8s.example.com
```

or:

```text
example.com
```

with an appropriate Kubernetes subdomain.

### Create a Route 53 Hosted Zone

If the organization domain is managed through Route 53, create a hosted zone for the Kubernetes DNS domain.

Example:

```bash
aws route53 create-hosted-zone \
  --name k8s.example.com \
  --caller-reference k8s-$(date +%s)
```

This creates the Route 53 hosted zone.

We can then use the corresponding DNS zone with KOPS.

> The exact DNS configuration depends on whether the organization already owns and manages the parent domain in Route 53 or another DNS provider.

### Using KOPS DNS Configuration

When creating the cluster, the DNS zone can be specified using KOPS configuration.

For example:

```bash
--dns-zone=k8s.example.com
```

For a learning environment, we can continue using:

```text
k8s.local
```

The important distinction is:

```text
Learning / Lab
    |
    |
k8s.local
```

```text
Production
    |
    |
Organization-controlled DNS
    |
    |
k8s.example.com
```

## 13. Create Kubernetes Cluster

Now we can create the KOPS cluster configuration.

### Learning Environment

For the learning environment, we can use:

```bash
kops create cluster \
  --name=demok8scluster.k8s.local \
  --state=s3://<unique-kops-state-store-name> \
  --zones=<aws-availability-zone> \
  --node-count=1 \
  --node-size=t2.micro \
  --master-size=t2.micro \
  --master-volume-size=8 \
  --node-volume-size=8
```

Example:

```bash
kops create cluster \
  --name=demok8scluster.k8s.local \
  --state=s3://kops-sachin-cluster-state-store \
  --zones=us-east-1a \
  --node-count=1 \
  --node-size=t2.micro \
  --master-size=t2.micro \
  --master-volume-size=8 \
  --node-volume-size=8
```

> **Important:** `t2.micro` is used here only for learning and demonstration. It should not be treated as a production sizing recommendation.

### Production DNS Example

For an organization-controlled DNS domain, the cluster name and DNS zone would be different.

For example:

```bash
kops create cluster \
  --name=k8s.example.com \
  --state=s3://<unique-kops-state-store-name> \
  --zones=<aws-availability-zone> \
  --dns-zone=k8s.example.com \
  --node-count=<worker-node-count> \
  --node-size=<worker-instance-type> \
  --master-size=<control-plane-instance-type>
```

The actual production values should be determined based on:

* Workload requirements
* Availability requirements
* AWS Region
* Number of Availability Zones
* Resource requirements
* Security requirements
* Cost considerations

## 14. Review Cluster Configuration

Before building the infrastructure, review the generated configuration.

Run:

```bash
kops edit cluster <cluster-name>
```

Example:

```bash
kops edit cluster demok8scluster.k8s.local
```

Review:

* Kubernetes version
* Instance types
* Number of nodes
* Availability Zones
* Networking
* DNS configuration
* Cluster settings

This step is important because KOPS may create multiple AWS resources.

> We should review the configuration before applying it.

## 15. Build the Kubernetes Cluster

Once the configuration has been reviewed, apply it.

```bash
kops update cluster \
  <cluster-name> \
  --state=s3://<unique-kops-state-store-name> \
  --yes
```

Example:

```bash
kops update cluster \
  demok8scluster.k8s.local \
  --state=s3://kops-sachin-cluster-state-store \
  --yes
```

KOPS will provision and configure the required AWS infrastructure.

Depending on the configuration, this may include:

* VPC resources
* Subnets
* Route tables
* Security groups
* EC2 instances
* IAM roles
* Control Plane components
* Worker Nodes
* Networking
* Certificates
* DNS-related resources

> The cluster may take several minutes to become fully operational.

## 16. Validate the Kubernetes Cluster

After the infrastructure has been created, validate the cluster:

```bash
kops validate cluster \
  <cluster-name> \
  --state=s3://<unique-kops-state-store-name>
```

Example:

```bash
kops validate cluster \
  demok8scluster.k8s.local \
  --state=s3://kops-sachin-cluster-state-store
```

> A successful validation indicates that the Kubernetes cluster is operational.

## 17. Verify Kubernetes Nodes

Check the Kubernetes nodes:

```bash
kubectl get nodes
```

Example:

```text
NAME                                        STATUS   ROLES
ip-10-0-1-25.us-east-1.compute.internal     Ready    <none>
```

Check all system Pods:

```bash
kubectl get pods -A
```

Check the current Kubernetes context:

```bash
kubectl config current-context
```

List all contexts:

```bash
kubectl config get-contexts
```

> At this point, we should have a functioning Kubernetes cluster.

## 18. Manage the KOPS Cluster Lifecycle

KOPS is not only used for initial cluster creation.

It also provides cluster lifecycle management.

### View Clusters

```bash
kops get cluster \
  --state=s3://<unique-kops-state-store-name>
```

### View Cluster Configuration

```bash
kops get cluster \
  <cluster-name> \
  --state=s3://<unique-kops-state-store-name>
```

### Edit Cluster

```bash
kops edit cluster \
  <cluster-name> \
  --state=s3://<unique-kops-state-store-name>
```

### Update Cluster

After modifying the configuration:

```bash
kops update cluster \
  <cluster-name> \
  --state=s3://<unique-kops-state-store-name> \
  --yes
```

### Validate Cluster

```bash
kops validate cluster \
  <cluster-name> \
  --state=s3://<unique-kops-state-store-name>
```

### Delete Cluster

When the cluster is no longer required:

```bash
kops delete cluster \
  <cluster-name> \
  --state=s3://<unique-kops-state-store-name> \
  --yes
```

## 19. Cost and Cleanup

KOPS creates and manages multiple AWS resources. These resources may incur AWS charges depending on the configuration and AWS account.

Resources may include:

* EC2 instances
* EBS volumes
* VPC resources
* Load Balancers
* S3 resources
* Route 53 resources

> For learning and temporary environments, we should clean up the resources after completing the exercise.

### Delete the Kubernetes Cluster

Use KOPS to delete the cluster and the AWS resources created for it:

```bash
kops delete cluster \
  <cluster-name> \
  --state=s3://<unique-kops-state-store-name> \
  --yes
```

Example:

```bash
kops delete cluster \
  demok8scluster.k8s.local \
  --state=s3://kops-sachin-cluster-state-store \
  --yes
```

> After deletion, verify that the expected AWS resources have been removed.

### Delete the KOPS State Store

> The S3 bucket used as the KOPS state store is separate from the Kubernetes infrastructure.

If the state store is no longer required, remove its contents and delete the bucket separately.

Remove the objects:

```bash
aws s3 rm s3://<unique-kops-state-store-name> --recursive
```

Delete the bucket:

```bash
aws s3api delete-bucket \
  --bucket <unique-kops-state-store-name> \
  --region <aws-region>
```

Verify:

```bash
aws s3 ls
```

### Cleanup Order

The recommended cleanup sequence is:

```text
Kubernetes Cluster
        |
        |
Delete KOPS Cluster
        |
        |
Verify AWS Resources
        |
        |
Delete KOPS State Store
        |
        |
Verify Cleanup
```

> **Important:** Do not delete the KOPS state store before deleting the cluster if KOPS still needs the state information to manage or remove the cluster.

## 20. Troubleshooting

### Issue 1: AWS AccessDenied

Error:

```text
AccessDenied
```

Verify:

```bash
aws sts get-caller-identity
```

Then verify that the AWS identity has the required permissions.

### Issue 2: KOPS Cannot Access State Store

Verify:

```bash
aws s3 ls s3://<unique-kops-state-store-name>
```

Also verify that the state store specified in the KOPS command is correct.

### Issue 3: Cluster Validation Failed

Run:

```bash
kops validate cluster \
  <cluster-name> \
  --state=s3://<unique-kops-state-store-name>
```

Check EC2 instances:

```bash
aws ec2 describe-instances
```

Check Kubernetes context:

```bash
kubectl config get-contexts
```

### Issue 4: kubectl Cannot Connect

Check the current context:

```bash
kubectl config current-context
```

List contexts:

```bash
kubectl config get-contexts
```

Verify that the expected KOPS cluster context is configured.

### Issue 5: DNS Problems

If using Route 53, verify the hosted zone:

```bash
aws route53 list-hosted-zones
```

Verify that the required DNS records and delegation are configured correctly.

## 21. Production Considerations

The commands above demonstrate the KOPS workflow. A real production deployment requires additional planning.

### High Availability

Production clusters should normally use:

* Multiple Control Plane nodes
* Multiple Worker Nodes
* Multiple Availability Zones

Example:

```text
                  AWS Region
                     |
                     |
          +----------+----------+
          |          |          |
          |          |          |
        AZ-A       AZ-B       AZ-C
          |          |          |
          |          |          |
       Control    Control    Control
        Plane      Plane      Plane
          |          |          |
          |          |          |
        Worker     Worker     Worker
         Node       Node       Node
```

### Production-Sized Instances

Do not use:

```text
t2.micro
```

for production workloads.

Instance sizing should be based on:

* CPU requirements
* Memory requirements
* Application workload
* Kubernetes system components
* Expected traffic

### Secure IAM

Avoid using unrestricted IAM policies for production.

Use:

* Least privilege
* Dedicated IAM roles
* Restricted permissions
* Short-lived credentials where possible

### Secure State Store

Protect the KOPS S3 state store using:

* Encryption
* Versioning
* Restricted IAM access
* Appropriate bucket policies
* Backup and recovery strategy

### Production DNS

Use an organization-controlled DNS domain rather than:

```text
k8s.local
```

For example:

```text
k8s.example.com
```

### Monitoring and Logging

Production Kubernetes environments should include:

* Metrics
* Monitoring
* Centralized logging
* Alerting
* Audit logging

### Backup and Disaster Recovery

Plan for:

* etcd/control-plane recovery
* Application data backup
* Persistent volume backup
* KOPS state-store protection
* Disaster recovery procedures

## 22. Official Documentation

The following documentation should be used as the primary reference when implementing KOPS and Kubernetes in real environments.

#### KOPS

[KOPS Official Documentation](https://kops.sigs.k8s.io/)

#### Kubernetes

[Kubernetes Official Documentation](https://kubernetes.io/docs/)

#### kubectl

[kubectl Official Documentation](https://kubernetes.io/docs/reference/kubectl/)

#### AWS CLI

[AWS CLI Official Documentation](https://docs.aws.amazon.com/cli/?)

#### Amazon EC2

[Amazon EC2 Documentation](https://docs.aws.amazon.com/ec2/?)

#### Amazon S3

[Amazon S3 Documentation](https://docs.aws.amazon.com/s3/)

#### Amazon Route 53

[Amazon Route 53 Documentation](https://docs.aws.amazon.com/route53/)

## Final Takeaways

In this setup, we learned how to provision a Kubernetes cluster on AWS using KOPS.

The complete workflow is:

```text
Management EC2
      |
      |
Install Tools
      |
      +--> Python 3
      +--> kubectl
      +--> AWS CLI
      +--> KOPS
      |
      |
Configure IAM
      |
      |
Configure AWS CLI
      |
      |
Create S3 State Store
      |
      |
Configure DNS
      |
      |
Create KOPS Cluster
      |
      |
Review Configuration
      |
      |
Build Infrastructure
      |
      |
Validate Cluster
      |
      |
Manage Cluster Lifecycle
      |
      |
Cleanup
```

The key production concept is that Kubernetes cluster management is a **lifecycle**, not a one-time installation.

```text
Provision
    |
    ↓
Configure
    |
    ↓
Operate
    |
    ↓
Upgrade
    |
    ↓
Scale
    |
    ↓
Troubleshoot
    |
    ↓
Decommission
```

KOPS provides the tooling required to automate much of this lifecycle for self-managed Kubernetes clusters on AWS.
