# 1.Redis Install on Ubuntu/Debian

```bash
sudo apt-get install lsb-release curl gpg
curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg
sudo chmod 644 /usr/share/keyrings/redis-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list
sudo apt-get update
sudo apt-get install redis -y
```

# Setting Redis `maxmemory` via Command Line
 
## Command
 
```bash
sudo sed -i 's/^#* *maxmemory .*/maxmemory 50mb/' /etc/redis/redis.conf
sudo systemctl restart redis-server
```
 
---
 
## How it Works
 
| Part | Description |
|------|-------------|
| `sudo` | Grants the necessary permissions to edit a system file. |
| `sed -i` | Tells the editor to change the file **in-place** (saving it directly). |
| `'s/.../.../` | This is the **substitute** command. |
| `^#* *maxmemory .*` | A regular expression that matches any line starting with an optional comment (`#`), followed by the word `maxmemory` and any existing value. |
| `maxmemory 50mb` | The replacement value — what the matched line is replaced with. |
 
---
# You can use the exact same sed logic to swap out the eviction policy. Since Redis defaults to noeviction in many versions, this command will either uncomment the line or update an existing setting to ensure it's explicitly set.

The Command
Run this to set your policy to noeviction:
```
sudo sed -i 's/^#* *maxmemory-policy .*/maxmemory-policy noeviction/' /etc/redis/redis.conf
sudo systemctl restart redis
```
# to  minitor memory

```
watch -n 1 "redis-cli info memory | grep used_memory_human"
```

Run this command to test:
redis-benchmark -q -n 200000 -d 1024 -t set -r 1000000
