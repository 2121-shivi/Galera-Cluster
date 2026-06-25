# Active-Active Galera Cluster using MariaDB and HAProxy

## Overview

This project demonstrates the implementation of an **Active-Active Galera Cluster** using **MariaDB** and **HAProxy** on Ubuntu virtual machines. The objective is to build a highly available database infrastructure where multiple database servers operate simultaneously and maintain synchronized data across all nodes.

Unlike a traditional single database server, an Active-Active Galera Cluster allows every database node to accept read and write requests. Any data inserted into one node is automatically replicated to the remaining nodes in real time. This architecture eliminates a single point of failure and improves database availability.

---

## Project Architecture

The project consists of **four virtual machines**.

| VM                    | IP Address  | Role                   |
| --------------------- | ----------- | ---------------------- |
| HAProxy Load Balancer | 192.168.0.2 | Load Balancer          |
| Galera DB Node 1      | 192.168.0.3 | Primary Bootstrap Node |
| Galera DB Node 2      | 192.168.0.4 | Cluster Member         |
| Galera DB Node 3      | 192.168.0.5 | Cluster Member         |

```
     <img width="981" height="643" alt="image" src="https://github.com/user-attachments/assets/ddbbdb6a-2a0d-4c1f-92eb-aa6aa5619b9c" />

           
```

---

## Technologies Used

* Ubuntu Server 22.04 LTS
* MariaDB Server
* Galera Cluster (galera-4)
* HAProxy
* SSH
* rsync

---

## Features

* Active-Active database architecture
* Synchronous data replication
* Three-node MariaDB Galera Cluster
* HAProxy load balancing
* High Availability (HA)
* Automatic failover support
* Fault tolerance
* Real-time data synchronization

---

## Project Workflow

1. Install MariaDB and Galera packages on all database nodes.
2. Configure each node with the same Galera cluster configuration.
3. Bootstrap the first database node to create the cluster.
4. Join the remaining database nodes to the cluster.
5. Configure HAProxy to distribute database traffic across all nodes.
6. Verify cluster health using `wsrep` status variables.
7. Test database replication by creating a database and inserting data.
8. Perform failover testing by stopping one database node and confirming that the remaining nodes continue serving requests.

---

## Testing Performed

The following validations were completed during the implementation:

* Successful cluster bootstrap
* DB2 and DB3 joined the cluster successfully
* Cluster size verification (`wsrep_cluster_size`)
* Cluster readiness verification (`wsrep_ready`)
* Real-time database replication
* HAProxy backend connectivity
* High availability and failover testing

---


## Learning Outcomes

This project provided hands-on experience with:

* MariaDB Galera Cluster
* Multi-master database clustering
* Database replication
* High Availability (HA)
* HAProxy load balancing
* Linux server administration
* SSH-based remote management
* Enterprise database deployment concepts

---

## Future Improvements

* Add Keepalived for HAProxy high availability.
* Configure SSL/TLS encryption for database communication.
* Integrate monitoring using Checkmk or Prometheus.
* Automate deployment using Ansible.
* Deploy the cluster in a cloud environment.

---

## Author

**Chand Parveen**
**Shivani Sharma**

B.Tech Computer Science & Engineering

Hands-on Project: Active-Active MariaDB Galera Cluster with HAProxy Load Balancer
