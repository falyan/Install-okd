![Logo](https://res.cloudinary.com/dabkaenvy/image/upload/v1762501262/Openshift_Logo_fj79vu.png)

# OKD INSTALLATION DOCUMENTATION

This document to guide step by step to install OKD version 4 specify [4.17.0-okd-scos.0.](https://github.com/okd-project/okd/releases/tag/4.17.0-okd-scos.0)


## 📚 Table of Content
- [Requirement for installation](#Requirement)
- [Topology](#struktur-proyek)
- [Instalasi](#instalasi)
- [Konfigurasi](#konfigurasi)
- [Menjalankan Aplikasi](#menjalankan-aplikasi)
- [Contoh Penggunaan](#contoh-penggunaan)
- [Deployment](#deployment)
- [Kontribusi](#kontribusi)
- [Lisensi](#lisensi)
- [Kontak](#kontak)


## Requirement for installation
This installation need requirement below.

- Resolver Record DNS
- 1 Bastion Server (Jumphost)
- 1 LoadBalancer Server (Haproxy)
- NTP server
- DHCP server
- 1 Node Bootstrap Server (Temporary)
- 3 Node Master (Control Plane)
- 6 Node Worker (Worker)

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
| worker4.midagri.gob.pe      | 10.200.106.43   | Storage 1 |
| worker5.midagri.gob.pe      | 10.200.106.44   | Storage 2 |
| worker6.midagri.gob.pe      | 10.200.106.45   | Storage 3 |

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
