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
