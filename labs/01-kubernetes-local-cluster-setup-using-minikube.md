
## 📄 `01-kubernetes-local-cluster-setup-using-minikube.md`

## Kubernetes Local Cluster Setup Using Minikube

**A Hands-On Guide to Installing and Managing a Local Kubernetes Cluster**

## Table of Contents

1. [Introduction](#1-introduction)
2. [Why We Need a Local Kubernetes Cluster](#2-why-we-need-a-local-kubernetes-cluster)
3. [Understanding Kubernetes Local Cluster Options](#3-understanding-kubernetes-local-cluster-options)
4. [What Is Minikube?](#4-what-is-minikube)
5. [Minikube Architecture Overview](#5-minikube-architecture-overview)
6. [Prerequisites for Minikube Installation](#6-prerequisites-for-minikube-installation)
7. [Understanding Hypervisors](#7-understanding-hypervisors)
8. [Installing Minikube](#8-installing-minikube)
   - [Linux Installation](#linux-installation)
   - [Windows Installation](#windows-installation)
   - [macOS Installation](#macos-installation)
9. [Installing kubectl](#9-installing-kubectl)
10. [Starting Kubernetes Cluster Using Minikube](#10-starting-kubernetes-cluster-using-minikube)
11. [Verifying Kubernetes Cluster](#11-verifying-kubernetes-cluster)
12. [Working With kubectl](#12-working-with-kubectl)
13. [Managing Minikube Cluster Lifecycle](#13-managing-minikube-cluster-lifecycle)
14. [Creating Multiple Kubernetes Clusters](#14-creating-multiple-kubernetes-clusters)
15. [Configuring Minikube Resources](#15-configuring-minikube-resources)
16. [Minikube Add-ons](#16-minikube-add-ons)
17. [Accessing Kubernetes Dashboard](#17-accessing-kubernetes-dashboard)
18. [Common Troubleshooting](#18-common-troubleshooting)
19. [Production Best Practices](#19-production-best-practices)
20. [Real-Time DevOps Usage](#20-real-time-devops-usage)
21. [Hands-On Lab Exercise](#21-hands-on-lab-exercise)

- [Final Takeaways](#final-takeaways)

## 1. Introduction

In this section, we will learn how to install a Kubernetes cluster on our local machine using **Minikube**.

In a real production environment, Kubernetes clusters are usually created using cloud platforms like:

* Amazon Elastic Kubernetes Service (EKS)
* Azure Kubernetes Service (AKS)
* Google Kubernetes Engine (GKE)

or using enterprise Kubernetes solutions like:

* OpenShift
* Rancher
* VMware Tanzu

However, creating a production-grade Kubernetes cluster requires infrastructure, networking, load balancers, multiple nodes, and additional configurations.

For learning Kubernetes, development, testing, and hands-on practice, creating a complete production cluster is not practical.

Therefore, we use a **local Kubernetes cluster**.

The most popular and beginner-friendly solution for running Kubernetes locally is:

> **Minikube**

In this guide, we will learn:

* What Minikube is
* How Minikube works
* How to install Minikube
* How to install kubectl
* How to create our first Kubernetes cluster
* How to manage the cluster
* How to use Minikube for Kubernetes learning and development

## 2. Why We Need a Local Kubernetes Cluster

### Problem Statement

Suppose we want to learn Kubernetes.

A Kubernetes production cluster normally requires:

* Multiple servers
* Control plane nodes
* Worker nodes
* Networking configuration
* Storage configuration
* Load balancers
* Security configurations

Creating this infrastructure only for learning is expensive and time-consuming.

For example:

A production Kubernetes cluster may require:

```
Control Plane Nodes
        |
        |
Worker Nodes
        |
        |
Application Pods
```

This requires multiple virtual machines or cloud resources.

### Solution: Local Kubernetes Cluster

A local Kubernetes cluster provides a lightweight Kubernetes environment running on our laptop or virtual machine.

Advantages:

* No cloud cost
* Easy installation
* Fast testing
* Perfect for learning
* Useful for application development

## 3. Understanding Kubernetes Local Cluster Options

There are multiple tools available to create local Kubernetes clusters.

Popular options:

| Tool                      | Purpose                                   |
| ------------------------- | ----------------------------------------- |
| Minikube                  | Beginner-friendly Kubernetes cluster      |
| Kind                      | Kubernetes inside Docker containers       |
| K3s                       | Lightweight Kubernetes distribution       |
| K3d                       | Runs K3s inside Docker                    |
| Docker Desktop Kubernetes | Kubernetes integrated with Docker Desktop |

### Why Minikube?

Minikube has been one of the most popular Kubernetes learning tools for many years.

Reasons:

* Official Kubernetes project
* Easy installation
* Supports multiple operating systems
* Supports Kubernetes features
* Supports add-ons
* Good community support

For beginners and Kubernetes labs, Minikube is one of the easiest choices.

## 4. What is Minikube?

Minikube is a tool that creates a single-node Kubernetes cluster on our local machine.

Official Website:

[Minikube Official Documentation](https://minikube.sigs.k8s.io/docs/?utm_source=chatgpt.com)

Normally Kubernetes requires multiple machines.

Minikube simplifies this by running:

```
+--------------------------------+
|        Developer Laptop        |
|                                |
|  +--------------------------+  |
|  |       Minikube Cluster   |  |
|  |                          |  |
|  |  Control Plane           |  |
|  |  Worker Node             |  |
|  |  Kubernetes Components   |  |
|  +--------------------------+  |
|                                |
+--------------------------------+
```

The entire Kubernetes environment runs inside a virtual machine or container.

## 5. Minikube Architecture Overview

A Minikube cluster contains:

### Control Plane Components

Responsible for Kubernetes management.

Includes:

* kube-apiserver
* etcd
* kube-controller-manager
* kube-scheduler

### Worker Node Components

Responsible for running applications.

Includes:

* kubelet
* kube-proxy
* Container Runtime

Architecture:

```
              kubectl
                 |
                 |
        Kubernetes API Server
                 |
     ----------------------------
     |                          |
 Controller Manager         Scheduler
     |
     |
    etcd


          Worker Node
     --------------------
     |                  |
 kubelet          Container Runtime
     |
 Application Pods
```

## 6. Prerequisites for Minikube Installation

Before installing Minikube, we need the following requirements.

### Hardware Requirements

Minimum:

| Resource | Requirement     |
| -------- | --------------- |
| CPU      | 2 CPUs          |
| Memory   | 2 GB RAM        |
| Storage  | 20 GB Free Disk |
| Internet | Required        |

Recommended for better experience:

| Resource | Recommended |
| -------- | ----------- |
| CPU      | 4 CPUs      |
| Memory   | 8 GB RAM    |
| Storage  | 30 GB+      |

## 7. Understanding Hypervisors

Minikube requires a driver to create the Kubernetes environment.

This driver is usually a hypervisor.

### What is a Hypervisor?

A hypervisor allows us to create and manage virtual machines.

Example:

```
Physical Laptop
       |
       |
 Hypervisor
       |
       |
 Virtual Machine
       |
       |
 Kubernetes Cluster
```

### Popular Hypervisors

#### Windows

Examples:

* Hyper-V
* VirtualBox
* VMware Workstation

#### Linux

Examples:

* KVM
* VirtualBox

#### macOS

Examples:

* HyperKit
* VirtualBox
* VMware Fusion

Minikube uses these drivers to create the Kubernetes node.

## 8. Installing Minikube

### Step 1: Install Minikube

Official installation guide:

[Install Minikube](https://minikube.sigs.k8s.io/docs/start/?utm_source=chatgpt.com)

#### Linux Installation

Download Minikube binary:

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
```

Install:

```bash
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

Verify:

```bash
minikube version
```

Expected output:

```
minikube version: v1.xx.x
```

#### Windows Installation

Using Chocolatey:

```powershell
choco install minikube
```

Verify:

```powershell
minikube version
```

#### macOS Installation

Using Homebrew:

```bash
brew install minikube
```

Verify:

```bash
minikube version
```

## 9. Installing kubectl

### What is kubectl?

kubectl is the Kubernetes command-line tool.

It allows us to communicate with the Kubernetes cluster.

Architecture:

```
Administrator
     |
     |
 kubectl
     |
     |
 Kubernetes API Server
     |
     |
 Kubernetes Cluster
```

kubectl can perform operations like:

* Create resources
* Delete resources
* Deploy applications
* View logs
* Debug applications

Official Documentation:

[Kubernetes kubectl Installation](https://kubernetes.io/docs/tasks/tools/?utm_source=chatgpt.com)

### Install kubectl on Linux

Download:

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

Install:

```bash
sudo install kubectl /usr/local/bin/kubectl
```

Verify:

```bash
kubectl version --client
```

## 10. Starting Kubernetes Cluster Using Minikube

Once Minikube is installed, create a cluster.

Basic command:

```bash
minikube start
```

Minikube will:

1. Download Kubernetes images
2. Create a virtual machine/container
3. Configure Kubernetes components
4. Configure kubectl access

Example output:

```
Starting control plane node minikube
Preparing Kubernetes environment
Configuring kubectl
Done! kubectl is now configured
```

## 11. Verifying Kubernetes Cluster

Check cluster status:

```bash
minikube status
```

Example:

```
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

Check nodes:

```bash
kubectl get nodes
```

Example:

```
NAME       STATUS   ROLES
minikube   Ready    control-plane
```

Check Kubernetes version:

```bash
kubectl version
```

## 12. Working With kubectl

### View Pods

```bash
kubectl get pods -A
```

### View Namespaces

```bash
kubectl get namespaces
```

### View Services

```bash
kubectl get services
```

### Describe Resources

Example:

```bash
kubectl describe node minikube
```

## 13. Managing Minikube Cluster Lifecycle

### Stop Cluster

When Kubernetes is not required:

```bash
minikube stop
```

This stops the cluster but keeps configuration.

### Start Cluster Again

```bash
minikube start
```

### Delete Cluster

To completely remove:

```bash
minikube delete
```

## 14. Creating Multiple Minikube Clusters

Minikube supports multiple clusters using profiles.

Example:

Create development cluster:

```bash
minikube start -p development
```

Create testing cluster:

```bash
minikube start -p testing
```

View profiles:

```bash
minikube profile list
```

Switch cluster:

```bash
kubectl config use-context development
```

## 15. Configuring Minikube Resources

We can customize:

* CPU
* Memory
* Disk
* Kubernetes version

Example:

```bash
minikube start \
--cpus=4 \
--memory=8192
```

This creates a cluster with:

* 4 CPUs
* 8GB RAM

## 16. Minikube Add-ons

Minikube supports many Kubernetes add-ons.

View available add-ons:

```bash
minikube addons list
```

Examples:

### Enable Dashboard

```bash
minikube addons enable dashboard
```

Open dashboard:

```bash
minikube dashboard
```

### Enable Ingress Controller

```bash
minikube addons enable ingress
```

### Enable Metrics Server

```bash
minikube addons enable metrics-server
```

## 17. Accessing Kubernetes Dashboard

Dashboard provides a graphical interface.

Command:

```bash
minikube dashboard
```

It opens:

* Nodes
* Pods
* Deployments
* Services
* Configurations

## 18. Common Troubleshooting

### Issue 1: Minikube Start Failed

Check:

```bash
minikube status
```

Restart:

```bash
minikube stop

minikube start
```

### Issue 2: Driver Problem

Check available drivers:

```bash
minikube start --help
```

Example:

```bash
minikube start --driver=docker
```

### Issue 3: kubectl Not Connected

Check context:

```bash
kubectl config current-context
```

Set Minikube context:

```bash
kubectl config use-context minikube
```

## 19. Production Best Practices

Even though Minikube is for development, follow production practices.

### Use kubectl Instead of Dashboard

Production engineers mostly use:

```bash
kubectl
```

because it supports:

* Automation
* CI/CD pipelines
* Scripting

### Maintain YAML Manifests

Avoid manually creating resources.

Store Kubernetes manifests:

```
kubernetes-manifests/

├── deployment.yaml
├── service.yaml
├── ingress.yaml
```

### Practice Production Tools

Use Minikube to practice:

* Helm
* Argo CD
* Prometheus
* Grafana
* Ingress
* Operators
* GitOps workflows

## 20. Real-Time DevOps Usage

Minikube can be used for:

### Application Testing

Example:

Deploy Spring Boot application:

```
Developer Code
      |
Docker Image
      |
Kubernetes Deployment
      |
Minikube Cluster
```

### Learning Kubernetes Objects

Practice:

* Pods
* ReplicaSets
* Deployments
* Services
* ConfigMaps
* Secrets
* Volumes

### Testing CI/CD Pipelines

Example:

```
GitHub
   |
Jenkins Pipeline
   |
Docker Build
   |
Kubernetes Deployment
   |
Minikube
```

## 21. Hands-On Lab Exercise

### Lab 1: Install Minikube

Task:

Install Minikube and verify version.

Command:

```bash
minikube version
```

### Lab 2: Create Kubernetes Cluster

Command:

```bash
minikube start
```

Verify:

```bash
kubectl get nodes
```

### Lab 3: Deploy Application

Create deployment:

```bash
kubectl create deployment nginx --image=nginx
```

Verify:

```bash
kubectl get deployments
```

Expose application:

```bash
kubectl expose deployment nginx --type=NodePort --port=80
```

Access:

```bash
minikube service nginx
```

## Final Takeaways

In this section, we learned:

* Why we need local Kubernetes clusters
* What Minikube is
* How Minikube works internally
* Hypervisor requirement
* Installing Minikube
* Installing kubectl
* Creating Kubernetes cluster
* Managing Minikube lifecycle
* Creating multiple profiles
* Configuring resources
* Using Minikube add-ons
* Practicing Kubernetes workloads locally

Minikube provides a simple and powerful way for DevOps engineers to practice Kubernetes without requiring expensive cloud infrastructure.

Once we are comfortable with Minikube, the same Kubernetes concepts can be applied to production platforms like:

* Amazon EKS
* Azure AKS
* Google GKE
* Enterprise Kubernetes platforms
