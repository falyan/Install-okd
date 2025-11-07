![Logo](https://res.cloudinary.com/dabkaenvy/image/upload/v1762501262/Openshift_Logo_fj79vu.png)

# OKD INSTALLATION DOCUMENTATION

This document to guide step by step to install OKD version 4 specify [4.17.0-okd-scos.0.](https://github.com/okd-project/okd/releases/tag/4.17.0-okd-scos.0)


## 📚 Table of Content
- [Requirement for install](#requirement)
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

| Hostname                  | IP Address      | Keterangan            |
|----------------------------|-----------------|------------------------|
| haproxy.okd.rnd.local      | 10.200.106.42   | HAProxy / Load Balancer |
| api.okd.rnd.local          | 10.200.106.42   | API endpoint eksternal |
| api-int.okd.rnd.local      | 10.200.106.42   | API internal cluster |
| *.apps.okd.rnd.local       | 10.200.106.42   | Wildcard route aplikasi |
| bootstrap.okd.rnd.local    | 10.200.106.41   | Node bootstrap sementara |
| bastion.okd.rnd.local      | 10.200.106.40   | Bastion / jumphost |
| master1.okd.rnd.local      | 10.200.106.34   | Control Plane node 1 |
| master2.okd.rnd.local      | 10.200.106.35   | Control Plane node 2 |
| master3.okd.rnd.local      | 10.200.106.36   | Control Plane node 3 |
| worker1.okd.rnd.local      | 10.200.106.37   | Worker node 1 |
| worker2.okd.rnd.local      | 10.200.106.38   | Worker node 2 |
| worker3.okd.rnd.local      | 10.200.106.39   | Worker node 3 |
| worker4.okd.rnd.local      | 10.200.106.43   | Worker node 4 |
| worker5.okd.rnd.local      | 10.200.106.44   | Worker node 5 |
| worker6.okd.rnd.local      | 10.200.106.45   | Worker node 6 |



