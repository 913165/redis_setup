
# Redis Cluster Setup on 3 WSL Nodes (6 Redis Instances)

This guide shows how to create a **Redis Cluster** using three Ubuntu WSL instances.

Cluster layout:

```
Node1 (WSL1)
 ├─ Redis 7000
 └─ Redis 7001

Node2 (WSL2)
 ├─ Redis 7002
 └─ Redis 7003

Node3 (WSL3)
 ├─ Redis 7004
 └─ Redis 7005
```

Total nodes: **6 (3 masters + 3 replicas)**

---

# 1 Install Redis (Run on ALL Nodes)

Run the following on **Node1, Node2, Node3**.

```bash
sudo apt-get install lsb-release curl gpg -y

curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg

sudo chmod 644 /usr/share/keyrings/redis-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list

sudo apt-get update
sudo apt-get install redis -y
```

Verify installation:

```bash
redis-server --version
```

---

# 2 Create Redis Configuration Template (Run on ALL Nodes)

```bash
cat <<EOF > redis.conf
bind 0.0.0.0
protected-mode no
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
EOF
```

---

# 3 Create Node Directories

## Node1 (WSL1)

```bash
mkdir 7000 7001
```

## Node2 (WSL2)

```bash
mkdir 7002 7003
```

## Node3 (WSL3)

```bash
mkdir 7004 7005
```

---

# 4 Copy redis.conf into Each Directory

## Node1

```bash
cp redis.conf 7000
cp redis.conf 7001
```

## Node2

```bash
cp redis.conf 7002
cp redis.conf 7003
```

## Node3

```bash
cp redis.conf 7004
cp redis.conf 7005
```

---

# 5 Set Correct Port for Each Redis Node

## Node1

```bash
echo "port 7000" >> 7000/redis.conf
echo "port 7001" >> 7001/redis.conf
```

## Node2

```bash
echo "port 7002" >> 7002/redis.conf
echo "port 7003" >> 7003/redis.conf
```

## Node3

```bash
echo "port 7004" >> 7004/redis.conf
echo "port 7005" >> 7005/redis.conf
```

---

# 6 Start Redis Nodes

## Node1

```bash
(cd 7000 && redis-server redis.conf &)
(cd 7001 && redis-server redis.conf &)
```

## Node2

```bash
(cd 7002 && redis-server redis.conf &)
(cd 7003 && redis-server redis.conf &)
```

## Node3

```bash
(cd 7004 && redis-server redis.conf &)
(cd 7005 && redis-server redis.conf &)
```

Verify processes:

```bash
ps -ef | grep redis
```

---

# 7 Create Redis Cluster

Run this command from **any one node**.

Replace the IP below if your WSL IP differs.

```bash
redis-cli --cluster create 172.27.122.70:7000 172.27.122.70:7001 172.27.122.70:7002 172.27.122.70:7003 172.27.122.70:7004 172.27.122.70:7005 --cluster-replicas 1
```

When prompted:

```
yes
```

Redis will automatically assign masters and replicas.

---

# 8 Verify Cluster Status

```bash
redis-cli -p 7000 cluster info
```

Expected output:

```
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
```

---

# 9 Check Cluster Nodes

```bash
redis-cli -p 7000 cluster nodes
```

---

# 10 Connect to Cluster

```bash
redis-cli -c -p 7000
```

Test routing:

```
set name redis
get name
```

You may see:

```
-> Redirected to slot located at another node
```

---

# Final Cluster Architecture

3 Masters
3 Replicas

Example:

Master 7000 → Replica 7003  
Master 7002 → Replica 7005  
Master 7004 → Replica 7001

---

# Redis Cluster Ready

Your **Redis Cluster across 3 WSL nodes** is now operational.
