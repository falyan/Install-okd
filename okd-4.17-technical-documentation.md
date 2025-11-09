<p align="center">
  <img src="https://res.cloudinary.com/dabkaenvy/image/upload/v1762501262/Openshift_Logo_fj79vu.png" alt="Cover" width="600"><br>
</p>

# OKD 4.17 - Technical Documentation Package

This document is guide step by step to install OKD version 4 specify [4.17.0-okd-scos.0.](https://github.com/okd-project/okd/releases/tag/4.17.0-okd-scos.0).


# 📚 Table of Content
- [INSTALLATION DOCUMENTATION](#installation-documentation)

## INSTALLATION DOCUMENTATION
- [Installation Method](#1-installation-method)
- [Prerequisites Checklist](#2-prerequisites-checklist)
- [Node Specification](#3-node-specification)
- [Topology](#4-topology)
- [Step by Step Procedure Installation](#5-step-by-step-procedure-installation)


### 1. Installation Method

 This installation uses the User Provisioned Infrastructure (UPI) method. User Provisioned Infrastructure (UPI) is an installation method in which the user is responsible for manually setting up and managing all the infrastructure components required for OKD, including virtual machines, networking, DNS, and load balancers. The OKD installer only provides the ignition configuration files, while the user handles the provisioning, configuration, and deployment of the cluster nodes.


### 2. Prerequisites Checklist
This installation need requirement below.


| Component                      | Description                                                                  |
| ------------------------------ | ---------------------------------------------------------------------------- |
| **DNS Resolver**               | A functional DNS record for internal and external name resolution.           |
| **Bastion Server (Jumphost)**  | Used as the main access point to deploy and manage OKD.                      |
| **Load Balancer (HAProxy)**    | Handles API and application routing for OKD control plane.                   |
| **NTP Server**                 | Ensures time synchronization across all cluster nodes.                       |
| **DHCP Server**                | Provides IP address assignment and static leases for cluster nodes.          |
| **Bootstrap Node (Temporary)** | Used during initial cluster deployment and removed afterward.                |
| **Master Nodes (3x)**          | Control plane nodes responsible for managing the OKD cluster.                |
| **Worker Nodes (3x)**          | Nodes where applications and workloads are deployed.                         |
| **Storage Nodes (3x)**         | Optional nodes for hosting persistent storage ( Cluster Ceph ).     |
| **Pull Secret**                | Required Red Hat or OKD pull secret to access and download container images. |


#### a. DNS Resolver

Configure the DNS server to provide name resolution for all cluster components, such as api.midagri.gob.pe, api-int.midagri.gob.pe, and node hostnames. Proper DNS configuration ensures that services can be correctly resolved within the cluster.


| Hostname and Domain                 | IP Address      | Describtion           |
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
| worker4.midagri.gob.pe      | 10.200.106.43   | worker node 4 |
| worker5.midagri.gob.pe      | 10.200.106.44   | worker node 5 |
| worker6.midagri.gob.pe      | 10.200.106.45   | worker node 6 |


#### b. Bastion Server (Jumphost)
In an OKD environment (Origin Community Distribution of Kubernetes, the open-source version of OpenShift), the bastion server plays a crucial role, especially during cluster setup and administration.
- **Secure gateway** for accessing all cluster nodes (master, worker).  
- **Central point for installation and administration** (`oc`, `kubectl`).  
- **Stores important configuration files** such as `kubeconfig` and `install-config.yaml`.  
- **Isolates the network** — only the bastion can SSH into cluster nodes.

<p align="center">
  <img src="https://res.cloudinary.com/dabkaenvy/image/upload/v1762506600/bastion.drawio_yzugrw.png" alt="OKD Bastion" width="600"><br>
  <em>Figure 1. OKD Bastion</em>
</p>

#### c. LoadBalancer (HAProxy)
HAProxy in an OKD environment acts as a load balancer that distributes incoming API and application traffic across the master and worker nodes.
It ensures high availability, failover, and reliable access to services like api.midagri.gob.pe, api-int.midagri.gob.pe, and *.apps.midagri.gob.pe.

#### d. NTP Server 
Deploy an NTP (Network Time Protocol) server and configure all nodes — Bastion, Bootstrap, Master, and Worker — to synchronize time. Accurate time synchronization prevents TLS and authentication issues during the installation, the NTP server source depends on your region. Makesure the ntp server running well and sync with your region, because the Control Plane IP will be assigned via DHCP. If your DHCP server does not provide time information, the control plane bootstrap process may fail.

#### e. DHCP Server
Configure a DHCP server to automatically assign Fix IP addresses with NIC and provide essential options such as gateway, DNS, and NTP servers. This setup simplifies the network configuration of CoreOS-based nodes during boot.

#### f. Node Bootstrap Server (Temporary)
The Bootstrap node in OKD is a temporary node used during cluster installation.
It initializes the control plane, deploys the first master components, and then becomes unnecessary after the cluster is fully up and running.

#### g. Master Node (Control Plane)
The Master nodes (Control Plane) manage and control the entire OKD cluster.
They handle API requests, scheduling, cluster state management, and communication between all components.

#### h. Worker Node (Wroker)
The Worker nodes run the actual workloads and applications in the OKD cluster.
They host pods, containers, and services deployed by users.

#### i. Storage Node (Persisten Volume)
The storage in this setup is organized as a Ceph cluster, providing distributed and highly available storage for the OKD environment. Initially, the storage nodes are configured as worker nodes, allowing them to participate in the OKD cluster while also hosting Ceph storage services. These storage nodes store persistent data used by applications, containers, and internal OKD services, ensuring data redundancy and reliability across the cluster.

#### j. Pull Secret 
The pull secret is required for OKD installation to authenticate and pull container images from the registry.

You can get it from the official OKD site:
👉 https://console.redhat.com/openshift/install/pull-secret

Login with your Red Hat account, then copy the JSON content and paste it into your install-config.yaml under the pullSecret field.

If you prefer to use a community, you can use an like this:

```bash
 { "auths": { "fake": { "auth": "aWQ6cGFzcwo=" } } }

```

### 3. Node Specification

For POC or development environments, each OKD 4.17 node should have at least the following specifications:

| Resource | Specification     |
| -------- | ----------------- |
| CPU      | 8 cores           |
| Memory   | 16 GB RAM         |
| Storage  | 120 GB disk space |


In production environments, the specifications should be scaled up based on the number of users, workloads, and the total capacity of applications running in the cluster to ensure optimal performance and stability.



| **Component**          | **Hostname / IP**         | **Operating System**                       | **CPU / RAM / Disk**            | **Main Services / Roles** |
|--------------------------|---------------------------|--------------------------------------------|----------------------------------|----------------------------|
| 🧩 **Bastion Server**    | `10.200.106.40`           | Rocky Linux 8.9 (Green Obsidian)           | 8 cores / 8 GB / 120 GB (sda)       | - openshift-installer<br>- NTP Server<br>- DNS Server<br>- Web Server (HTTP)<br>- OC & Kubectl Client<br>- Jump Host |
| ⚙️ **Load Balancer**     | `10.200.106.42`           | Ubuntu 24.04.3 LTS (Noble)                 | 8 cores / 16 GB / 120 GB (sda)        | - HAProxy<br> :6443 → API Server (masters)<br> :22623 → Machine Config Server<br> :80 / :443 → Ingress Routers<br>- DHCP Server |
| 🚀 **Bootstrap Server**  | `10.200.106.41`           | CentOS Stream CoreOS 417.9.2024            | 8 cores / 16 GB / 120 GB (sda)         | - Bootstrap Ignition<br>- Temporary Control Plane<br>- OC & Kubectl Client |
| 🧭 **Master Nodes**      | `10.200.106.34–36`        | CentOS Stream CoreOS 417.9.2024            | 8 cores / 16 GB / 120 GB each (sda)    | - Control Plane (etcd, API, scheduler, controller) |
| 💼 **Worker Nodes**      | `10.200.106.37–39`        | CentOS Stream CoreOS 417.9.2024            | 8 cores / 8 GB / 120 GB each (sda)    | - Compute / Application Workloads |
| 💾 **Storage Nodes**     | `10.200.106.43–45` *(example)* | CentOS Stream CoreOS 417.9.2024        | 8 cores / 16 GB / 120 GB each (sda) and 300 GB each (sdb)    | - Ceph Cluster (distributed worker, persistent volumes) |

### 4. Topology
<p align="center">
  <img src="https://res.cloudinary.com/dabkaenvy/image/upload/v1762509569/20251107_1659_image_ajy0xi.png" alt="OKD Topology" width="600"><br>
  <em>Figure 2. OKD Topology</em>
</p>



### 5. Step by Step Procedure Installation
In this installation phase, we will perform the following steps to deploy and configure the OKD 4.17 cluster environment:

- [Network Routing](#network-routing)
- [Create OKD Manifest](#create-okd-manifest)
- [Setting http Web Server](#setting-http-web-server)
- [Installation with Bootstraping](#installation-with-bootstraping)

#### a. Network Routing

1. **DHCP Setup** 
Configure dynamic IP address allocation and network boot, DHCP Configuration will install on haproxy server. This document provides a general overview and DHCP configuration for the OKD cluster network.The setup uses a /24 subnet (10.200.106.0/24) with static IP assignments for all OKD components.

   **Note:**
   The DHCP server can be deployed as a stand-alone service, but if sufficient resources are available, it is recommended to host it on an existing infrastructure node.In this guide, the DHCP service will be installed on the HAProxy server for simplicity and efficient resource usage.

<p align="center">
  <img src="https://res.cloudinary.com/dabkaenvy/image/upload/v1762676155/Screenshot_2025-11-09_at_15.15.28_xvdglg.png" alt="Network-Topology" width="400"><br>
  <em>Figure 3. OKD Network Topology</em>
</p>


| Role      | Hostname                 | IP Address    | MAC Address (Example) |
| --------- | ------------------------ | ------------- | --------------------- |
| Bastion   | bastion.midagri.gob.pe   | 10.200.106.40 | 50:6b:8d:9a:11:bb     |
| Bootstrap | bootstrap.midagri.gob.pe | 10.200.106.41 | 50:6b:8d:9a:11:aa     |
| Master 1  | master1.midagri.gob.pe   | 10.200.106.34 | 50:6b:8d:9a:11:c1     |
| Master 2  | master2.midagri.gob.pe   | 10.200.106.35 | 50:6b:8d:9a:11:c2     |
| Master 3  | master3.midagri.gob.pe   | 10.200.106.36 | 50:6b:8d:9a:11:c3     |
| Worker 1  | worker1.midagri.gob.pe   | 10.200.106.37 | 50:6b:8d:9a:11:d1     |
| Worker 2  | worker2.midagri.gob.pe   | 10.200.106.38 | 50:6b:8d:9a:11:d2     |
| Worker 3  | worker3.midagri.gob.pe   | 10.200.106.39 | 50:6b:8d:9a:11:d3     |
| Worker 4  | worker4.midagri.gob.pe   | 10.200.106.43 | 50:6b:8d:9a:11:d4     |
| Worker 5  | worker5.midagri.gob.pe   | 10.200.106.44 | 50:6b:8d:9a:11:d5     |
| Worker 6  | worker6.midagri.gob.pe   | 10.200.106.45 | 50:6b:8d:9a:11:d6     |


#### 🧾 Explanation

- **Subnet:** 10.200.106.0/24  
- **Gateway:** 10.200.106.1  
- **Bastion Host:** Provides DNS and NTP services  
- **DHCP Server:** Runs on Ubuntu (10.200.106.42)  
- **Dynamic Range:** 10.200.106.50–100  
- **Static IPs:** Reserved for OKD components  
- **MAC Addresses:** Example only — replace with real hardware values  
 

2. **DNS & NTP Configuration** 
   Set up name resolution and time synchronization for all cluster nodes.
Both the DNS and NTP services are installed on the Bastion server (10.200.106.40) to ensure consistent hostname resolution and clock synchronization across the environment.

#### b. Create OKD Manifest   

1.  **Ignition File Generation**
   Create ignition files for the bootstrap, master, and worker nodes on bastion server

#### c.Setting http Web Server
   Install a lightweight web server such as Nginx or Apache to host the Ignition configuration files required during the OKD installation. These Ignition files (bootstrap.ign, master.ign, worker.ign) are fetched by each node during boot using an Ignition URL.

   Example Ignition URLs used in boot parameters:

   ```bash
   http://10.200.106.40/bootstrap.ign
   http://10.200.106.40/master.ign
   http://10.200.106.40/worker.ign
   ```

#### d. Installation with Bootstraping   

1. **Bootstrap Installation**
   Initialize the control plane and start the cluster on Bootstrap server fot first installation process.

2. **Master Worker & worker Node Joining**
   Join remaining nodes to complete the OKD cluster setup.