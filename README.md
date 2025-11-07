![Logo](https://res.cloudinary.com/dabkaenvy/image/upload/v1762501262/Openshift_Logo_fj79vu.png)

# OKD INSTALLATION DOCUMENTATION

This document is guide step by step to install OKD version 4 specify [4.17.0-okd-scos.0.](https://github.com/okd-project/okd/releases/tag/4.17.0-okd-scos.0)


## 📚 Table of Content
- [Requirement for installation](#requirement-for-installation)
- [Node Specification and Topology ](#node-specification-and-topology)
- [Installation](#installation)


## Requirement for installation
This installation need requirement below.

- Resolver Record DNS
- 1 Bastion Server (Jumphost)
- 1 LoadBalancer Server (Haproxy)
- NTP server
- DHCP server
- 1 Node Bootstrap Server (Temporary)
- 3 Node Master (Control Plane)
- 3 Worker Node
- 3 Storage Node

### Resolver Record DNS

Configure the DNS server to provide name resolution for all cluster components, such as api.midagri.gob.pe, api-int.midagri.gob.pe, and node hostnames. Proper DNS configuration ensures that services can be correctly resolved within the cluster.

| Hostname                  | IP Address      | Describtion           |
|----------------------------|-----------------|------------------------|
| haproxy.midagri.gob.pe      | 10.200.106.42   | HAProxy / Load Balancer |
| api.midagri.gob.pe          | 10.200.106.42   | API endpoint eksternal |
| api-int.midagri.gob.pe      | 10.200.106.42   | API internal cluster |
| *.apps.midagri.gob.pe       | 10.200.106.42   | Wildcard route aplikasi |
| bootstrap.midagri.gob.pe    | 10.200.106.41   | Node bootstrap sementara |
| bastion.midagri.gob.pe      | 10.200.106.40   | Bastion / jumphost |
| master1.midagri.gob.pe      | 10.200.106.34   | Control Plane node 1 |
| master2.midagri.gob.pe      | 10.200.106.35   | Control Plane node 2 |
| master3.midagri.gob.pe      | 10.200.106.36   | Control Plane node 3 |
| worker1.midagri.gob.pe      | 10.200.106.37   | Worker node 1 |
| worker2.midagri.gob.pe      | 10.200.106.38   | Worker node 2 |
| worker3.midagri.gob.pe      | 10.200.106.39   | Worker node 3 |
| storage1.midagri.gob.pe      | 10.200.106.43   | Storage 1 |
| storage2.midagri.gob.pe      | 10.200.106.44   | Storage 2 |
| storage3.midagri.gob.pe      | 10.200.106.45   | Storage 3 |

### Bastion Server (Jumphost)
In an OKD environment (Origin Community Distribution of Kubernetes, the open-source version of OpenShift), the bastion server plays a crucial role, especially during cluster setup and administration.
- **Secure gateway** for accessing all cluster nodes (master, worker).  
- **Central point for installation and administration** (`oc`, `kubectl`).  
- **Stores important configuration files** such as `kubeconfig` and `install-config.yaml`.  
- **Isolates the network** — only the bastion can SSH into cluster nodes.

![Bastion](https://res.cloudinary.com/dabkaenvy/image/upload/v1762506600/bastion.drawio_yzugrw.png)

### LoadBalancer (Haproxy)
HAProxy in an OKD environment acts as a load balancer that distributes incoming API and application traffic across the master and worker nodes.
It ensures high availability, failover, and reliable access to services like api.midagri.gob.pe, api-int.midagri.gob.pe, and *.apps.midagri.gob.pe.

### NTP Server 
Deploy an NTP (Network Time Protocol) server and configure all nodes — Bastion, Bootstrap, Master, and Worker — to synchronize time. Accurate time synchronization prevents TLS and authentication issues during the installation, the NTP server source depends on your region. Makesure the ntp server running well and sync with your region, because the Control Plane IP will be assigned via DHCP. If your DHCP server does not provide time information, the control plane bootstrap process may fail.

### DHCP Server
Configure a DHCP server to automatically assign Fix IP addresses with NIC and provide essential options such as gateway, DNS, and NTP servers. This setup simplifies the network configuration of CoreOS-based nodes during boot.

### Node Bootstrap Server (Temporary)
The Bootstrap node in OKD is a temporary node used during cluster installation.
It initializes the control plane, deploys the first master components, and then becomes unnecessary after the cluster is fully up and running.

### Master Node (Control Plane)
The Master nodes (Control Plane) manage and control the entire OKD cluster.
They handle API requests, scheduling, cluster state management, and communication between all components.

### Worker Node (Wroker)
The Worker nodes run the actual workloads and applications in the OKD cluster.
They host pods, containers, and services deployed by users.

### Storage Node
The Storage nodes form a Ceph cluster that provides distributed and highly available storage for the OKD environment.
They store persistent data used by applications, containers, and internal OKD services.

# Node Specification and Topology
## 🧱 OKD 4.17 Cluster Node Specification

| **Component**          | **Hostname / IP**         | **Operating System**                       | **CPU / RAM / Disk**            | **Main Services / Roles** |
|--------------------------|---------------------------|--------------------------------------------|----------------------------------|----------------------------|
| 🧩 **Bastion Server**    | `10.200.106.40`           | Rocky Linux 8.9 (Green Obsidian)           | 8 cores / 8 GB / 120 GB (sda)       | - openshift-installer<br>- NTP Server<br>- DNS Server<br>- Web Server (HTTP)<br>- OC & Kubectl Client<br>- Jump Host |
| ⚙️ **Load Balancer**     | `10.200.106.42`           | Ubuntu 24.04.3 LTS (Noble)                 | 8 cores / 16 GB / 120 GB (sda)        | - HAProxy<br> :6443 → API Server (masters)<br> :22623 → Machine Config Server<br> :80 / :443 → Ingress Routers<br>- DHCP Server |
| 🚀 **Bootstrap Server**  | `10.200.106.41`           | CentOS Stream CoreOS 417.9.2024            | 8 cores / 16 GB / 120 GB (sda)         | - Bootstrap Ignition<br>- Temporary Control Plane<br>- OC & Kubectl Client |
| 🧭 **Master Nodes**      | `10.200.106.34–36`        | CentOS Stream CoreOS 417.9.2024            | 8 cores / 16 GB / 120 GB each (sda)    | - Control Plane (etcd, API, scheduler, controller) |
| 💼 **Worker Nodes**      | `10.200.106.37–39`        | CentOS Stream CoreOS 417.9.2024            | 32 cores / 32 GB / 120 GB each (sda)    | - Compute / Application Workloads |
| 💾 **Storage Nodes**     | `10.200.106.43–45` *(example)* | CentOS Stream CoreOS 417.9.2024        | 8 cores / 16 GB / 120 GB each (sda) and 300 GB each (sdb)    | - Ceph Cluster (distributed storage, persistent volumes) |

![Topology](https://res.cloudinary.com/dabkaenvy/image/upload/v1762509569/20251107_1659_image_ajy0xi.png)

# Installation
In this installation phase, we will perform the following steps to deploy and configure the OKD 4.17 cluster environment:

1. [**DHCP Setup** ](#dhcp-setup) 
   Configure dynamic IP address allocation and network boot.

2. [**DNS & NTP Configuration**](#dns-&-ntp-configuration) 
   Set up name resolution and time synchronization for all cluster nodes.

3. [**Ignition File Generation**](#ignition-file-generation)
   Create ignition files for the bootstrap, master, and worker nodes ojn bastion server

4. [**Bootstrap Installation**](#bootstrap-installation) 
   Initialize the control plane and start the cluster installation process.

5. [**Master Worker & Storage Node Joining**](#master-worker-&-storage-node-joining) 
   Join remaining nodes to complete the OKD cluster setup.