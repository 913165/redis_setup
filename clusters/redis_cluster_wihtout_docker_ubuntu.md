# Redis Install on Ubuntu/Debian 

```bash
sudo apt-get install lsb-release curl gpg
curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg
sudo chmod 644 /usr/share/keyrings/redis-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list
sudo apt-get update
sudo apt-get install redis -y
```
# Redis will start automatically, and it should restart at boot time. If Redis doesn't start across reboots, you may need to manually enable it:
```bash
sudo systemctl enable redis-server
sudo systemctl start redis-server
sudo systemctl stop redis-server
sudo systemctl disable redis-server
```

# create redis.conf into current directory

```bash
cat <<EOF > redis.conf
port 7000
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
EOF
```

# give permission
```
chmod -R 777 *
```

# Redis Cluster Setup (Ports 7000–7008)

This guide contains all commands used to successfully create and verify a Redis Cluster locally.

---

# 1. Create directories for Redis nodes

```bash
mkdir 700{0..8}
```

---

# 2. Copy redis.conf to all folders

```bash
for i in 700*; do cp -v redis.conf $i; done
```

---

# 3. Update port in each redis.conf based on folder name

```bash
for i in 700*; do sed -i "s/^port .*/port $i/" $i/redis.conf; done
```

Example config inside each folder:

```
port 7001
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
```

---

# 4. Start Redis servers for all nodes

```bash
for i in 700*; do (cd $i && redis-server redis.conf &); done
```

---

# 5. Verify Redis servers are running

```bash
ps -ef | grep redis
```

Expected output example:

```
redis-server *:7001 [cluster]
redis-server *:7002 [cluster]
redis-server *:7003 [cluster]
...
```

---

# 6. Create Redis Cluster

```bash
redis-cli --cluster create \
127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 \
127.0.0.1:7005 127.0.0.1:7006 127.0.0.1:7007 127.0.0.1:7008 \
--cluster-replicas 1
```

When prompted type:

```
yes
```

---

# 7. Verify cluster health

```bash
redis-cli -p 7001 cluster info
```

Expected important fields:

```
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_known_nodes:8
cluster_size:4
```

---

# 8. Check cluster nodes

```bash
redis-cli -p 7001 cluster nodes
```

---

# 9. Connect in cluster mode

```bash
redis-cli -c -p 7001
```

Test cluster routing:

```
set name redis
get name
```

You may see redirection like:

```
-> Redirected to slot located at another node
```

---

# 10. Verify slot coverage

```bash
redis-cli --cluster check 127.0.0.1:7001
```

Expected result:

```
[OK] All 16384 slots covered.
```

---

# Final Cluster Architecture

```
4 Masters
4 Replicas
Total nodes = 8
```

Example structure:

```
Master 7001 -> Replica 7005
Master 7002 -> Replica 7006
Master 7003 -> Replica 7007
Master 7004 -> Replica 7008
```

---

# Redis Cluster Ready

Your Redis Cluster is now fully operational.
).
