# Ubuntu-proxy-update-automation

lets say you have a VPN/PROXY listening on this address:
```
10.125.248.133.7890
```
the system wants to know the answer of this question:
> Which programs should use this proxy, and how do they know

you have to add the cat /etc/environment file:
```bash
sudo vim cat /etc/environment
```
and the file contains the proxy ip and port like this:
```bash
HTTP_PROXY=http://127.0.0.1:7890
HTTPS_PROXY=http://127.0.0.1:7890
NO_PROXY=localhost,127.0.0.1,::1
```
