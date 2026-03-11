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
```

---

# Redis Sentinel Setup (Fresh Ubuntu VM)

This README contains the full set of commands to install Redis, configure a master with replicas, configure Redis Sentinel, and verify that failover works.

---

# Architecture

Master      : 6379
Replica 1   : 6380
Replica 2   : 6381

Sentinel 1  : 26379
Sentinel 2  : 26380
Sentinel 3  : 26381

---

# 1. Install Redis

```bash
sudo apt update
sudo apt install redis-server -y
```

Check version

```bash
redis-server --version
```

---

# 2. Create directories

```bash
mkdir redis-master redis-replica1 redis-replica2 sentinel1 sentinel2 sentinel3
```

---

# 3. Create Redis Master configuration

```bash
cat <<EOF > redis-master/redis.conf
port 6379
bind 127.0.0.1
protected-mode no
appendonly yes
EOF
```

---

# 4. Create Replica 1 configuration

```bash
cat <<EOF > redis-replica1/redis.conf
port 6380
replicaof 127.0.0.1 6379
appendonly yes
protected-mode no
EOF
```

---

# 5. Create Replica 2 configuration

````bash
cat <<EOF > redis-replica2/redis.conf
port 6381
replicaof 127.0.0.1 6379
appendonly yes
protected-mode no
EOF

---

# 6. Start Redis servers

```bash
redis-server redis-master/redis.conf &
redis-server redis-replica1/redis.conf &
redis-server redis-replica2/redis.conf &
````

Verify

```bash
ps -eo pid,cmd | grep redis-server
```

Expected output

```
redis-server *:6379
redis-server *:6380
redis-server *:6381
```

---

# 7. Create Sentinel configuration

## Sentinel 1

```bash
cat <<EOF > sentinel1/sentinel.conf
port 26379
sentinel monitor mymaster 127.0.0.1 6379 2
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 10000
sentinel parallel-syncs mymaster 1
EOF
```

## Sentinel 2

```bash
cat <<EOF > sentinel2/sentinel.conf
port 26380
sentinel monitor mymaster 127.0.0.1 6379 2
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 10000
sentinel parallel-syncs mymaster 1
EOF
```

## Sentinel 3

```bash
cat <<EOF > sentinel3/sentinel.conf
port 26381
sentinel monitor mymaster 127.0.0.1 6379 2
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 10000
sentinel parallel-syncs mymaster 1
EOF
```

---

# 8. Start Sentinel nodes

```bash
redis-server sentinel1/sentinel.conf --sentinel &
redis-server sentinel2/sentinel.conf --sentinel &
redis-server sentinel3/sentinel.conf --sentinel &
```

Verify

```bash
ps -eo pid,cmd | grep redis-server
```

Expected

```
redis-server *:6379
redis-server *:6380
redis-server *:6381
redis-server *:26379 [sentinel]
redis-server *:26380 [sentinel]
redis-server *:26381 [sentinel]
```

---

# 9. Verify replication

```bash
redis-cli -p 6379 INFO replication
```

Expected

```
role:master
connected_slaves:2
slave0:ip=127.0.0.1,port=6380,state=online
slave1:ip=127.0.0.1,port=6381,state=online
```

---

# 10. Check Sentinel information

Check master

```bash
redis-cli -p 26379 SENTINEL masters
```

Check replicas

```bash
redis-cli -p 26379 SENTINEL replicas mymaster
```

Check sentinels

```bash
redis-cli -p 26379 SENTINEL sentinels mymaster
```

---

# 11. Test automatic failover

Stop the master

```bash
redis-cli -p 6379 shutdown
```

Sentinel will automatically promote one replica to master.

Check again

```bash
redis-cli -p 26379 SENTINEL masters
```

---

# Final Topology

```
        Sentinel 1
            |
Sentinel 2 --- Sentinel 3

            |
        Redis Master (6379)
          /           \
Replica (6380)    Replica (6381)
```

---

# Redis Sentinel Environment Ready

Your Redis Sentinel setup with automatic failover is now fully operational.
