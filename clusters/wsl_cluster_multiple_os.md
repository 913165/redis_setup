
# Redis Cluster Reset / Delete Guide

This guide explains how to remove or reset a Redis Cluster **without shutting down the Redis nodes**.

It works for clusters created using **Redis Cluster mode**.

---

# 1. Verify Current Cluster Status

Run on any node:

```bash
redis-cli -p 7000 cluster info
```

Example output:

```
cluster_state:ok
cluster_slots_assigned:16384
cluster_known_nodes:6
```

---

# 2. Reset Cluster Configuration on Each Node

Run the following command on **every Redis node**.

Example:

```bash
redis-cli -p 7000 cluster reset
```

Repeat for all nodes:

```bash
redis-cli -p 7001 cluster reset
redis-cli -p 7002 cluster reset
redis-cli -p 7003 cluster reset
redis-cli -p 7004 cluster reset
redis-cli -p 7005 cluster reset
```

This removes:

- cluster relationships
- slot assignments
- cluster metadata

Each node becomes a **standalone Redis server** again.

---

# 3. If Nodes Contain Data (Use Hard Reset)

If Redis refuses the reset because keys exist, run:

```bash
redis-cli -p 7000 cluster reset hard
```

Repeat for all nodes:

```bash
redis-cli -p 7001 cluster reset hard
redis-cli -p 7002 cluster reset hard
redis-cli -p 7003 cluster reset hard
redis-cli -p 7004 cluster reset hard
redis-cli -p 7005 cluster reset hard
```

This will:

- remove all cluster state
- clear slot assignments
- detach nodes from cluster

---

# 4. Remove Cluster Metadata Files (Optional but Recommended)

Redis stores cluster metadata in `nodes.conf`.

Delete these files:

```bash
rm 700*/nodes.conf
```

Or individually:

```bash
rm 7000/nodes.conf
rm 7001/nodes.conf
rm 7002/nodes.conf
rm 7003/nodes.conf
rm 7004/nodes.conf
rm 7005/nodes.conf
```

---

# 5. Verify Nodes Are No Longer in Cluster

Run:

```bash
redis-cli -p 7000 cluster info
```

Expected output:

```
cluster_state:fail
cluster_known_nodes:1
```

This means the node is **standalone**.

---

# 6. Create a New Cluster (Optional)

If you want to recreate the cluster:

```bash
redis-cli --cluster create \
172.27.122.70:7000 \
172.27.122.70:7001 \
172.27.122.70:7002 \
172.27.122.70:7003 \
172.27.122.70:7004 \
172.27.122.70:7005 \
--cluster-replicas 1
```

When prompted type:

```
yes
```

---

# Summary

To delete/reset a Redis cluster without shutting down nodes:

```
redis-cli -p PORT cluster reset hard
```

Run on **all nodes**.

---

Your Redis nodes are now **detached from the cluster and ready for reuse**.
