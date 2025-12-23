# Ubuntu-proxy-update-automation

lets say you have a **VPN/PROXY** listening on this address:
```
10.125.248.133:7890
```
the system wants to know the answer of this question:
> Which programs should use this proxy, and how do they know

There’s no single standard. Each layer of the system has its own method.

in order to answer I have this approach:

## 1. ```/etc/proxy.conf```
- It does nothing by itself, the file just notes what is the ip and port that your **VPN/PROXY** is listening on

for example:
```bash
HTTP_PROXY=http://127.0.0.1:7890
HTTPS_PROXY=http://127.0.0.1:7890
NO_PROXY=localhost,127.0.0.1,::1
```

## 2. ```/etc/profile.d/http_proxy.sh```
**For user terminal programs (bash/zsh)**
When you log in:
- bash / zsh
- Terminal
- curl
- wget
- git (most of the time)
This file is executed.

But the problem is things like **Docker**, **apt**, and **systemd** don’t care about the file

the file content should be sth like:
```bash
source /etc/proxy.conf
export http_proxy="http://$PROXY_IP"
export https_proxy="http://$PROXY_IP"
export HTTP_PROXY="http://$PROXY_IP"
export HTTPS_PROXY="http://$PROXY_IP"
```

## 3. ```/etc/environment```
**System-wide environment variables (but dumb)**

This file is read:
- By the login manager
- By some sessions
But the problem still exists ...
this does not support variable expantion and you shlould hard write the values like:
```bash
HTTP_PROXY=http://10.125.248.113:7890
HTTPS_PROXY=http://10.125.248.113:7890
NO_PROXY=localhost,127.0.0.1,::1
```

## 4. ```/etc/apt/apt.conf.d/95proxies```
apt:
- Does not correctly read env variables
- Not bash
- Not systemd

**apt has its own syntax like:**
```
Acquire::http::Proxy "http://10.125.248.113:7890";
```
the file contetnt should be sth like:
```APT
Acquire::http::Proxy "http://10.125.248.113:7890/";
Acquire::https::Proxy "http://10.125.248.113:7890/";
```
- apt itself can't udrestands the env, and it has its own config undrestood! so you have to hard write it in the file.
  
 
## 5. ```cat /etc/systemd/system/docker.service.d/http-proxy.conf```
**Docker**
using the prixy by the docker daemon that is runnig by the systemd
the content of file looks like this:
```bash
[Service]
Environment="HTTP_PROXY=http://10.125.248.113:7890"
Environment="HTTPS_PROXY=http://10.125.248.113:7890"
Environment="NO_PROXY=localhost,127.0.0.1,::1"
```
**docker deomon is runnig before the user login, "systemd" so it apears not to undrestans the env and expands it. so agin like the APT you have to hard write it in the file!**

## 6. /usr/local/bin/pupdate
and for the final part we have the automation
this is the bash script for it:

```bash
#!/bin/sh

# --- ask user ---
printf "Proxy IP: "
read PROXY_IP

printf "Proxy Port: "
read PROXY_PORT

PROXY_URL="http://$PROXY_IP:$PROXY_PORT"

# --- write /etc/proxy.conf ---
cat <<EOF > /etc/proxy.conf
HTTP_PROXY=$PROXY_URL
HTTPS_PROXY=$PROXY_URL
NO_PROXY=localhost,127.0.0.1,::1
EOF

# --- write /etc/environment ---
cat <<EOF > /etc/environment
HTTP_PROXY=$PROXY_URL
HTTPS_PROXY=$PROXY_URL
NO_PROXY=localhost,127.0.0.1,::1
EOF

# --- apt ---
cat <<EOF > /etc/apt/apt.conf.d/95proxies
Acquire::http::Proxy "$PROXY_URL/";
Acquire::https::Proxy "$PROXY_URL/";
EOF

# --- docker ---
DOCKER_DIR="/etc/systemd/system/docker.service.d"
mkdir -p "$DOCKER_DIR"

cat <<EOF > "$DOCKER_DIR/http-proxy.conf"
[Service]
Environment="HTTP_PROXY=$PROXY_URL"
Environment="HTTPS_PROXY=$PROXY_URL"
Environment="NO_PROXY=localhost,127.0.0.1,::1"
EOF

systemctl daemon-reexec
systemctl restart docker

echo "Proxy updated to $PROXY_URL"

```
 and then give it the permission:
 ```bash
sudo chmod +x /usr/local/bin/pupdate
```

---
after this you can run ```sudo pupdate``` to change the ip and port!





