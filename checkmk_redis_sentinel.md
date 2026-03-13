# Monitoring Redis Cluster with CheckMK Raw on Ubuntu (WSL2)

This guide documents the exact steps to successfully monitor a Redis cluster using CheckMK Raw Edition on Ubuntu WSL2, where both CheckMK and Redis are running on the same machine.

---

## Prerequisites

- Ubuntu (WSL2 or native)
- CheckMK Raw Edition installed (version 2.4.0p23 used here)
- Redis cluster running (this guide uses ports 7000–7008)

---

## Step 1 — Verify Your Redis Cluster is Running

Before touching CheckMK, confirm your Redis cluster nodes are up.

```bash
ps aux | grep redis-server
```

Expected output shows all cluster nodes:
```
root  21491  redis-server *:7000 [cluster]
root  21494  redis-server *:7001 [cluster]
...
root  21515  redis-server *:7008 [cluster]
```

Also test connectivity:
```bash
redis-cli -p 7000 ping    # should return PONG
redis-cli -p 7000 cluster info | head -5
```

---

## Step 2 — Install the CheckMK Agent

The CheckMK agent collects data from your machine and sends it to the CheckMK server.

```bash
# The .deb package is already bundled with CheckMK
sudo dpkg -i /opt/omd/versions/2.4.0p23.cre/share/check_mk/agents/check-mk-agent*.deb

# Enable and start the agent socket
sudo systemctl enable check-mk-agent.socket
sudo systemctl start check-mk-agent.socket

# Verify it is running
sudo systemctl status check-mk-agent.socket
```

Confirm the agent responds:
```bash
check_mk_agent | head -5
```

---

## Step 3 — Deploy the Redis Plugin

CheckMK uses a **bash plugin** called `mk_redis` to collect Redis metrics. It ships with CheckMK but must be manually deployed to the agent plugins directory.

```bash
# Create the plugins directory if it does not exist
sudo mkdir -p /usr/lib/check_mk_agent/plugins

# Copy the plugin from the CheckMK installation
sudo cp /opt/omd/versions/2.4.0p23.cre/share/check_mk/agents/plugins/mk_redis \
        /usr/lib/check_mk_agent/plugins/

# Make it executable
sudo chmod +x /usr/lib/check_mk_agent/plugins/mk_redis
```

---

## Step 4 — Configure the Redis Plugin

The `mk_redis` plugin is a **bash script** and uses a specific bash-style config format (NOT Python dict format).

```bash
# Create the config directory
sudo mkdir -p /etc/check_mk

# Write the config file with your cluster nodes
sudo tee /etc/check_mk/mk_redis.cfg << 'EOF'
REDIS_INSTANCES=(master replica1 replica2)

REDIS_HOST_master="127.0.0.1"
REDIS_PORT_master="6379"

REDIS_HOST_replica1="127.0.0.1"
REDIS_PORT_replica1="6380"

REDIS_HOST_replica2="127.0.0.1"
REDIS_PORT_replica2="6381"
EOF
```

> ⚠️ **Important:** The config format is bash variable style, NOT Python. Using Python dict format (`instances = [...]`) will be silently ignored by the plugin.

Verify the config looks correct:
```bash
cat /etc/check_mk/mk_redis.cfg
```

---

## Step 5 — Test the Plugin

Run the plugin manually to confirm it collects data from your cluster.

```bash
sudo /usr/lib/check_mk_agent/plugins/mk_redis
```

You should see output like:
```
<<<redis_info:sep(58)>>>
[[[127.0.0.1:7000]]]
# Server
redis_version:8.6.1
redis_mode:cluster
cluster_enabled:1
role:master
...
[[[127.0.0.1:7001]]]
...
```

If you see only the internal CheckMK Redis (unix-socket, `tcp_port:0`), the config is not being read — go back to Step 4.

If you get a Python import error:
```bash
pip3 install redis --break-system-packages
```

---

## Step 6 — Add the Host in CheckMK UI

Open your CheckMK web interface and add the machine as a monitored host.

1. Navigate to **Setup → Hosts → Add Host**
2. Set **Hostname** to your machine's IP (e.g. `172.27.122.70`)
3. Set **IPv4 address** to the same IP
4. Leave **Monitoring agents** as default (`Checkmk agent`)
5. Click **Save & run service discovery**

To find your machine's IP:
```bash
hostname -I
```

---

## Step 7 — Discover and Accept Redis Services

After saving the host, CheckMK runs a service discovery. You will see a list of discovered services.

1. Scroll through the list to find all **Redis nodeXXXX** services
2. Click **"Accept all"** to add all services to monitoring
3. Click **"Activate Changes"** (orange button, top right)

Each Redis node will have **3 services** monitored:
| Service | What it shows |
|---|---|
| `Redis node7000 Clients` | Number of connected clients |
| `Redis node7000 Persistence` | RDB/AOF save status |
| `Redis node7000 Server Info` | Mode, uptime, version, port |

---

## Step 8 — View Live Monitoring

After activation, navigate to:
```
Monitor → Overview → All Hosts → 172.27.122.70 → Services
```

All services will initially show **PEND** (grey) — this is normal. After 1–2 minutes they will turn **OK** (green) as the first checks complete.

To speed this up:
```
Commands → Reschedule active checks
```

---

## Troubleshooting

| Problem | Cause | Fix |
|---|---|---|
| Plugin shows only unix-socket Redis | Config not being read | Check bash format in `/etc/check_mk/mk_redis.cfg` |
| `Agent plug-ins: 0` in discovery | Plugin not deployed | Redo Step 3 |
| `Connection refused` on ports | Redis not running | Run `redis-server` cluster nodes |
| No Redis in service discovery | Plugin not producing output | Run plugin manually (Step 5) |
| Services stuck on PEND | First check not run yet | Wait 2 mins or reschedule checks |
| Python import error | Missing redis library | `pip3 install redis --break-system-packages` |

---

## File Locations Summary

| File | Purpose |
|---|---|
| `/opt/omd/versions/2.4.0p23.cre/share/check_mk/agents/plugins/mk_redis` | Source plugin (do not edit) |
| `/usr/lib/check_mk_agent/plugins/mk_redis` | Deployed plugin (agent reads this) |
| `/etc/check_mk/mk_redis.cfg` | Plugin config (your cluster nodes) |
| `/usr/lib/check_mk_agent/` | Agent plugins directory |

---

## Result

Once complete, CheckMK monitors all 9 Redis cluster nodes (ports 7000–7008) with live metrics for clients, persistence, and server health — all showing **OK** status in the dashboard.
