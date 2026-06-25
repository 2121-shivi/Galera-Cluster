

# Active-Active Galera Cluster Simulation Setup Guide

## Project Overview

This project demonstrates the implementation of an Active-Active Galera Cluster using MariaDB and HAProxy. The cluster consists of three MariaDB database nodes and one HAProxy load balancer to provide high availability, synchronous replication, and fault tolerance.

---

# Environment Details

| VM Name | IP Address | Role |
|---------|------------|------|
| HAProxyLoadBalancer | 192.168.0.2 | Load Balancer |
| GaleraDBNode1 | 192.168.0.3 | Database Node 1 |
| GaleraDBNode2 | 192.168.0.4 | Database Node 2 |
| GaleraDBNode3 | 192.168.0.5 | Database Node 3 |

---

# Step 1: Verify Network Connectivity

Run the following commands on all virtual machines.

```bash
ip addr
ping 8.8.8.8
ping google.com
```

Purpose:
- Verify IP address.
- Verify internet connectivity.
- Verify DNS resolution.

---

# Step 2: Update Ubuntu Servers

Run on DB1, DB2, and DB3.

```bash
apt update
apt upgrade -y
```

Purpose:
- Update package repositories.
- Install latest security updates.

---

# Step 3: Install MariaDB and Galera

Run on all database nodes.

```bash
apt install -y mariadb-server galera-4 rsync
```

Purpose:
- MariaDB provides database services.
- Galera provides synchronous replication.
- Rsync transfers database state.

---

# Step 4: Stop MariaDB Service

```bash
systemctl stop mariadb
```

Purpose:
- Prevent MariaDB from starting before configuration.

---

# Step 5: Configure Galera on DB1

VM: **GaleraDBNode1 (192.168.0.3)**

Edit configuration:

```bash
nano /etc/mysql/mariadb.conf.d/60-galera.cnf
```

Add:

```ini
[mysqld]
bind-address=0.0.0.0

binlog_format=ROW
default_storage_engine=InnoDB
innodb_autoinc_lock_mode=2

wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so

wsrep_cluster_name=galera_cluster
wsrep_cluster_address=gcomm://192.168.0.3,192.168.0.4,192.168.0.5

wsrep_node_address=192.168.0.3
wsrep_node_name=db1

wsrep_sst_method=rsync
```

---

# Step 6: Configure DB2

VM: **GaleraDBNode2 (192.168.0.4)**

```ini
wsrep_node_address=192.168.0.4
wsrep_node_name=db2
```

---

# Step 7: Configure DB3

VM: **GaleraDBNode3 (192.168.0.5)**

```ini
wsrep_node_address=192.168.0.5
wsrep_node_name=db3
```

---

# Step 8: Configure Firewall

Run on all database nodes.

```bash
ufw allow 3306/tcp
ufw allow 4567/tcp
ufw allow 4567/udp
ufw allow 4568/tcp
ufw allow 4444/tcp
ufw allow 2232/tcp
ufw reload
```

Purpose:
- Open Galera and database ports.

---

# Step 9: Bootstrap the Cluster

Run only on DB1.

```bash
galera_new_cluster
```

Verify:

```bash
systemctl status mariadb
```

---

# Step 10: Verify Cluster Size

Run on DB1.

```bash
mysql -e "SHOW STATUS LIKE 'wsrep_cluster_size';"
```

Expected:

```text
wsrep_cluster_size = 1
```

---

# Step 11: Join DB2

Run on DB2.

```bash
systemctl start mariadb
```

Verify:

```bash
mysql -e "SHOW STATUS LIKE 'wsrep_cluster_size';"
```

Expected:

```text
wsrep_cluster_size = 2
```

---

# Step 12: Join DB3

Run on DB3.

```bash
systemctl start mariadb
```

Verify:

```bash
mysql -e "SHOW STATUS LIKE 'wsrep_cluster_size';"
```

Expected:

```text
wsrep_cluster_size = 3
```

---

# Step 13: Verify Cluster Status

```bash
mysql -e "SHOW STATUS LIKE 'wsrep_cluster_status';"
```

Expected:

```text
Primary
```

Check readiness:

```bash
mysql -e "SHOW STATUS LIKE 'wsrep_ready';"
```

Expected:

```text
ON
```

---

# Step 14: Install HAProxy

Run on HAProxyLoadBalancer.

```bash
apt update
apt install -y haproxy mariadb-client
```

---

# Step 15: Configure HAProxy

Edit:

```bash
nano /etc/haproxy/haproxy.cfg
```

Add:

```cfg
frontend galera_frontend
    bind *:3306
    default_backend galera_backend

backend galera_backend
    balance roundrobin
    option tcp-check

    server db1 192.168.0.3:3306 check
    server db2 192.168.0.4:3306 check
    server db3 192.168.0.5:3306 check
```

---

# Step 16: Start HAProxy

```bash
systemctl restart haproxy
systemctl enable haproxy
systemctl status haproxy
```

---

# Step 17: Create Test Database

Run on DB1.

```bash
mysql
```

```sql
CREATE DATABASE projectdb;

USE projectdb;

CREATE TABLE students(
id INT PRIMARY KEY AUTO_INCREMENT,
name VARCHAR(50)
);

INSERT INTO students(name)
VALUES('Chand');

SELECT * FROM students;
```

---

# Step 18: Verify Replication

DB2:

```bash
mysql -e "SELECT * FROM projectdb.students;"
```

DB3:

```bash
mysql -e "SELECT * FROM projectdb.students;"
```

Expected:

```text
+----+-------+
| id | name  |
+----+-------+
| 1  | Chand |
+----+-------+
```

---

# Step 19: Failover Testing

Stop DB1.

```bash
systemctl stop mariadb
```

Check DB2:

```bash
mysql -e "SHOW STATUS LIKE 'wsrep_cluster_size';"
```

Expected:

```text
wsrep_cluster_size = 2
```

---

# Step 20: Recover Failed Node

```bash
systemctl start mariadb
```

Verify:

```bash
mysql -e "SHOW STATUS LIKE 'wsrep_cluster_size';"
```

Expected:

```text
wsrep_cluster_size = 3
```

---

# SSH Access

```bash
ssh -p 2232 root@192.168.0.2
ssh -p 2232 root@192.168.0.3
ssh -p 2232 root@192.168.0.4
ssh -p 2232 root@192.168.0.5
```

---

# Final Verification Commands

```bash
mysql -e "SHOW STATUS LIKE 'wsrep_cluster_size';"

mysql -e "SHOW STATUS LIKE 'wsrep_ready';"

mysql -e "SHOW STATUS LIKE 'wsrep_cluster_status';"
```

Expected:

```text
wsrep_cluster_size = 3
wsrep_ready = ON
wsrep_cluster_status = Primary
```

---

# Project Outcome

- Active-Active database cluster successfully created.
- Real-time database replication implemented.
- High availability achieved.
- Database failover successfully tested.
- HAProxy load balancing configured.
- Enterprise database architecture simulated.

---

# Author

**Chand Perveen**  
**Shivani Sharma**


Project: **Active-Active Galera Cluster Simulation**
