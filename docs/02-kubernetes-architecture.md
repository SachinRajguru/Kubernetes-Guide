
## 📄 `02-kubernetes-architecture.md`

## Kubernetes Architecture — Understanding the Control Plane and Worker Nodes

## Table of Contents

1. [Introduction](#1-introduction)
2. [Why Is Kubernetes Called K8s?](#2-why-is-kubernetes-called-k8s)
3. [Recap: Docker vs Kubernetes](#3-recap-docker-vs-kubernetes)
4. [Why Kubernetes Architecture Is Important](#4-why-kubernetes-architecture-is-important)
5. [Kubernetes Architecture Overview](#5-kubernetes-architecture-overview)
6. [Docker Container Lifecycle vs Kubernetes Pod Lifecycle](#6-docker-container-lifecycle-vs-kubernetes-pod-lifecycle)
7. [Worker Node (Data Plane) Components](#7-worker-node-data-plane-components)
   - [kubelet](#kubelet)
   - [kube-proxy](#kube-proxy)
   - [Container Runtime](#container-runtime)
8. [Control Plane Components](#8-control-plane-components)
   - [API Server](#api-server)
   - [Scheduler](#scheduler)
   - [etcd](#etcd)
   - [Controller Manager](#controller-manager)
   - [Cloud Controller Manager](#cloud-controller-manager)
9. [Complete Pod Creation Workflow](#9-complete-pod-creation-workflow)
10. [End-to-End Request Flow](#11-end-to-end-request-flow)
11. [Interview Questions](#12-interview-questions)

- [Key Takeaways](#key-takeaways)

## 1. Introduction

In the previous section, we answered one of the most important beginner questions:

> Why do we need Kubernetes when Docker already exists?

We learned that Docker is excellent for creating and running containers, but it lacks several production-grade capabilities such as clustering, self-healing, automatic scaling, and enterprise integrations.

Now that we understand *why Kubernetes exists*, the next logical step is to understand **how Kubernetes works internally**.

That is exactly what we will cover in this section.

## 2. Why Is Kubernetes Called K8s?

Before diving into the architecture, let's answer a fun question that almost every Kubernetes beginner asks.

**Why is Kubernetes abbreviated as K8s?**

The word **Kubernetes** contains:

* First letter: **K**
* Last letter: **s**
* Eight letters between them:

```text
K u b e r n e t e s
^                 ^
K                 s
```

There are **8 letters** between **K** and **s**.

Therefore:

```text
K + 8 + s = K8s
```

This style of abbreviation is called a **numeronym**, where the number represents the count of omitted letters.

Examples:

* Kubernetes → K8s
* Internationalization → I18N
* Localization → L10N

Although K8s is commonly used in documentation and industry conversations, both **Kubernetes** and **K8s** refer to the same platform.

## 3. Recap: Docker vs Kubernetes

Before understanding Kubernetes architecture, we must clearly understand the difference between Docker and Kubernetes.

From the previous section, we learned that Kubernetes solves four major limitations of Docker.

| Docker Limitation           | Kubernetes Solution       |
| --------------------------- | ------------------------- |
| Single Host Architecture    | Cluster Architecture      |
| No Auto Healing             | Self-Healing              |
| No Auto Scaling             | Horizontal Scaling        |
| Limited Enterprise Features | Production-Ready Platform |

These four capabilities directly influence Kubernetes' architecture.

Unlike Docker, Kubernetes requires multiple components working together to provide these enterprise features.

## 4. Why Kubernetes Architecture Is Important

Many tutorials simply list Kubernetes components such as:

* API Server
* Scheduler
* etcd
* Controller Manager
* kubelet
* kube-proxy

While these definitions are technically correct, they often do not explain **why** these components exist or how they collaborate.

A better approach is to compare Kubernetes with Docker.

By understanding how Docker creates a container and how Kubernetes creates a Pod, every Kubernetes component begins to make sense naturally.

That is the approach followed throughout this guide.

## 5. Kubernetes Architecture Overview

A Kubernetes cluster is divided into two major parts.

```text
               Kubernetes Cluster

           +-----------------------+
           |     Control Plane     |
           +-----------------------+
                       |
       -----------------------------------
      |                |                  |
+-----------+    +-----------+     +-----------+
| Worker 1  |    | Worker 2  | ... | Worker N  |
+-----------+    +-----------+     +-----------+
```

The cluster consists of:

* **Control Plane (Master Node)** — Makes decisions and manages the cluster.
* **Worker Nodes (Data Plane)** — Execute workloads by running application Pods.

Think of it like an organization:

* The **Control Plane** is the management team.
* The **Worker Nodes** are the employees executing the assigned work.

## 6. Docker Container Lifecycle vs Kubernetes Pod Lifecycle

To understand Kubernetes architecture, let's first understand how Docker starts a container.

## Docker

Suppose we execute:

```bash
docker run nginx
```

The process is simplified as follows:

```text
User
   │
docker run
   │
Docker Engine
   │
Container Runtime
   │
Container Starts
```

A container cannot run by itself.

It requires a **container runtime**, which is responsible for:

* Creating containers
* Managing container processes
* Interacting with the Linux kernel

Historically, Docker used a runtime known as **Docker Shim**.

## Kubernetes

Creating a Pod is similar but involves additional management components.

```text
User
   │
kubectl apply
   │
API Server
   │
Worker Node
   │
kubelet
   │
Container Runtime
   │
Pod Starts
```

Because Kubernetes manages applications across multiple machines, additional coordination is required.

This coordination is handled by the Control Plane.

## 7. Worker Node (Data Plane) Components

Worker Nodes are responsible for executing application workloads.

Every Worker Node contains three major components.

```text
+--------------------------------------+
|          Worker Node                 |
|                                      |
|  kubelet                             |
|  kube-proxy                          |
|  Container Runtime                   |
+--------------------------------------+
```

### 7.1 kubelet

The **kubelet** is the primary agent running on every Worker Node.

Its responsibilities include:

* Receiving instructions from the Control Plane.
* Creating Pods.
* Monitoring Pod health.
* Reporting node and Pod status.
* Restarting failed workloads when instructed.

Think of kubelet as the **node manager**.

Without kubelet, Pods cannot be managed.

### 7.2 Container Runtime

Pods contain one or more containers.

Containers require a runtime to execute.

Kubernetes does **not** require Docker specifically.

It supports any runtime that implements the **Container Runtime Interface (CRI)**.

Common runtimes include:

* containerd
* CRI-O
* Docker Shim (historically)

This flexibility allows organizations to choose the runtime that best suits their infrastructure.

### 7.3 kube-proxy

Applications running inside Pods need networking.

kube-proxy provides networking functionality such as:

* Assigning network rules.
* Enabling Pod communication.
* Implementing Service networking.
* Providing basic load balancing.
* Managing traffic routing using Linux networking (such as iptables or IPVS, depending on configuration).

Without kube-proxy, Pods would not communicate efficiently within the cluster.

## 8. Control Plane Components

The Control Plane is responsible for managing the entire Kubernetes cluster.

Its major components are:

```text
+-------------------------------------------+
|             Control Plane                 |
|-------------------------------------------|
| API Server                                |
| Scheduler                                 |
| etcd                                      |
| Controller Manager                        |
| Cloud Controller Manager (Optional)       |
+-------------------------------------------+
```

### 8.1 API Server

The **API Server** is the heart of Kubernetes.

Every request enters the cluster through the API Server.

Examples:

* Creating Pods
* Updating Deployments
* Scaling applications
* Deleting resources

All Kubernetes clients communicate with the API Server.

Examples include:

* kubectl
* Kubernetes Dashboard
* CI/CD pipelines
* Automation tools

The API Server validates requests before processing them.

### 8.2 Scheduler

The Scheduler decides **where** a Pod should run.

Suppose a cluster contains three Worker Nodes.

```text
Node A
Node B
Node C
```

When a Pod is created, the Scheduler evaluates factors such as:

* Available CPU
* Available Memory
* Resource requests
* Node conditions
* Scheduling constraints

It then selects the most appropriate Worker Node.

The Scheduler **does not create the Pod**.

It only decides **which node should host it**.

### 8.3 etcd

Every Kubernetes cluster requires a reliable data store.

That data store is **etcd**.

etcd stores cluster information such as:

* Nodes
* Pods
* Services
* ConfigMaps
* Secrets
* Cluster state
* Metadata

etcd is a distributed **key-value database**.

It acts as the source of truth for the Kubernetes cluster.

If etcd is unavailable, the Control Plane cannot reliably manage the cluster.

### 8.4 Controller Manager

Kubernetes continuously tries to maintain the **desired state** of the cluster.

Examples:

* Maintain the required number of Pod replicas.
* Replace failed Pods.
* Monitor Node health.
* Manage endpoints.

These responsibilities are handled by Kubernetes controllers.

The **Controller Manager** runs and manages these built-in controllers.

For example, if a Deployment specifies:

```text
Replicas = 3
```

and one Pod fails,

the Controller Manager ensures another Pod is created so that the desired state of **three Pods** is maintained.

### 8.5 Cloud Controller Manager

The Cloud Controller Manager is used when Kubernetes runs on cloud platforms.

Examples include:

* Amazon EKS
* Azure AKS
* Google GKE

Its responsibilities include interacting with cloud-specific services such as:

* Load Balancers
* Block Storage
* Node lifecycle
* Cloud networking

Instead of Kubernetes implementing cloud-specific logic for every provider, each cloud provider integrates through the Cloud Controller Manager.

If Kubernetes is deployed entirely on-premises, this component may not be required.

## 9. Complete Pod Creation Workflow

The following sequence summarizes how a Pod is created.

```text
Developer
    │
kubectl apply
    │
API Server
    │
Scheduler selects Worker Node
    │
kubelet receives instruction
    │
Container Runtime starts containers
    │
kube-proxy configures networking
    │
Pod becomes Ready
```

This illustrates how multiple Kubernetes components collaborate to deploy an application.

## 10. End-to-End Request Flow

The complete request flow can be summarized as follows.

```text
User
  │
  │ kubectl apply
  ▼
API Server
  │
  ▼
Scheduler
  │
  ▼
Worker Node
  │
  ▼
kubelet
  │
  ▼
Container Runtime
  │
  ▼
Pod Running
  │
  ▼
kube-proxy
  │
  ▼
Application Accessible
```

This flow represents the lifecycle of a Pod from deployment to execution.

## 11. Interview Questions

### Q1. What are the two major parts of a Kubernetes cluster?

**Answer**

* Control Plane
* Worker Nodes (Data Plane)

### Q2. What is the responsibility of kubelet?

**Answer**

The kubelet runs on every Worker Node, creates Pods, monitors their health, and communicates with the Control Plane.

### Q3. What is the purpose of kube-proxy?

**Answer**

kube-proxy manages networking for Pods and Services, enabling communication and basic load balancing.

### Q4. Why does Kubernetes need a Scheduler?

**Answer**

The Scheduler determines the most suitable Worker Node for a new Pod based on resource availability and scheduling policies.

### Q5. What is etcd?

**Answer**

etcd is Kubernetes' distributed key-value datastore that stores the cluster's configuration and current state.

### Q6. What is the role of the API Server?

**Answer**

The API Server is the entry point for all Kubernetes operations. Every request to create, update, or delete resources passes through it.

### Q7. What is the purpose of the Controller Manager?

**Answer**

It runs Kubernetes controllers that continuously reconcile the actual cluster state with the desired state, such as maintaining the correct number of Pod replicas.

### Q8. When is the Cloud Controller Manager used?

**Answer**

It is used when Kubernetes runs on cloud platforms to integrate with cloud services such as load balancers, storage, and networking.

## Key Takeaways

* Kubernetes architecture is divided into the **Control Plane** and **Worker Nodes**.
* Worker Nodes execute workloads using **kubelet**, **kube-proxy**, and a **Container Runtime**.
* The Control Plane manages the cluster through the **API Server**, **Scheduler**, **etcd**, **Controller Manager**, and **Cloud Controller Manager**.
* Every Kubernetes request starts with the API Server and ends with Pods running on Worker Nodes.
* Understanding the architecture provides the foundation for learning advanced Kubernetes concepts.

## Next Section

**[Kubernetes Cluster Provisioning with KOPS on AWS](03-kubernetes-cluster-provisioning-with-kops-on-aws.md)**

*Kubernetes in Production — Cluster Lifecycle Management, Kubernetes Distributions, and Production Installation*