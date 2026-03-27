# Install Guide

__Table of Contents__
+ [NOTE](#note)
+ [PRE-REQUIETIES](#pre-requieties)
+ [CONFIGURE USER POLICY](#configure-user-policy)
+ [ENABLE DOCKER TO AUTO-BOOT](#enable-docker-to-auto-boot)
+ [CONFIG HTTP PROXY (IF NEEDED)](#config-http-proxy-if-needed)
+ [TRY DOCKER](#try-docker)

## NOTE

This is the guide for __WSL2__ with __Ubuntu__ distro. Before you start,
check your WSL distro version with
`wsl --list -v`, and it will show the __VERSION__ with __2__
if correct.

And to start `systemd` of WSL2, just edit the `/etc/wsl.conf`,
and add the configuration bellow. (You can use `sudo vim /etc/wsl.conf`)
```
[boot]
systemd=true
```
[Ref: wsl official dev-blog](https://devblogs.microsoft.com/commandline/systemd-support-is-now-available-in-wsl/)

After the configuring, terminate the WSL2 server with `wsl --shutdown` ( or `wsl --terminate <your distro name>`)
and restart it. Then you can check if the systemd is running with
```bash
systemctl --no-pager status user.slice > /dev/null 2>&1 && echo 'OK: Systemd is running' || echo 'FAIL: Systemd not running'
```

## PRE-REQUIETIES
Install pre-requieties for docker:

```bash
sudo apt update && sudo apt upgrade
sudo apt install --no-install-recommends apt-transport-https ca-certificates curl gnupg2
```

__IMPORTANT:__ And then configure the iptables:

```bash
sudo update-alternatives --set iptables /usr/sbin/iptables-legacy
sudo update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
```

## INSTALL DOCKER

Now we can install docker.

First we need to update the apt source for docker.

```bash
. /etc/os-release # load env vars form os-release
curl -fsSL https://download.docker.com/linux/${ID}/gpg | sudo tee /etc/apt/trusted.gpg.d/docker.asc
echo "deb [arch=amd64] https://download.docker.com/linux/${ID} ${VERSION_CODENAME} stable" | sudo tee /etc/apt/sources.list.d/docker.list
sudo apt update
```

And then just install it with apt:
```bash
sudo apt install docker-ce docker-ce-cli containerd.io
```

## CONFIGURE USER POLICY

We need add the current user in docker group manually.
```bash
sudo usermod -aG docker $USER
```

To confirm the change worked, close the terminal tab and open a new Ubuntu tab, then run:
```bash
groups | grep docker
```
You should see `docker` in the content. And then terminate the WSL2 again (`wsl --shutdown`).

## ENABLE DOCKER TO AUTO-BOOT

Finally, use:
```bash
sudo systemctl enable docker.service
```
or:
``` bash
sudo service docker start
```

It can be checked via:
```bash
systemctl list-units --type=service
```
And you should see `docker.service` is not red, and is with `active` and `running` states.

## CONFIG HTTP PROXY (IF NEEDED)

If you need add proxy for docker daemon, just add the Systemd configure:

```bash
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo vim /etc/systemd/system/docker.service.d/http-proxy.conf
```
and add these lines:

```toml
[Service] 
Environment="HTTP_PROXY=http://<Your Proxy IP or Domain>:<port>" 
Environment="HTTPS_PROXY=http://<Your Proxy IP or Domain>:<port>"
```

Then restart your Docker service with new config:

```bash
sudo systemctl daemon-reload 
sudo systemctl restart docker 
```

And check the new env var with:

```bash
sudo systemctl show --property=Environment docker
```
You should see something like:
```bash
Environment=HTTP_PROXY=http://<Your Proxy IP or Domain>:<port> HTTPS_PROXY=http://<Your Proxy IP or Domain>:<port>
```


## TRY DOCKER

You can try docker to see if it can connect to
the dockerd:
```bash
docker run --rm hello-world
```

Refs:
+ https://docs.docker.com/config/daemon/systemd/#httphttps-proxy
+ https://dataedo.com/docs/installing-docker-on-windows-via-wsl
+ https://dev.to/klo2k/run-docker-in-wsl2-in-5-minutes-via-systemd-without-docker-desktop-28gi