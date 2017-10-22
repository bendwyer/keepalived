# keepalived
Collected instructions for setting up keepalived with HAProxy on Ubuntu 16.04.

## Table of Contents
- [Background](#background)
- [Basic Server Information](#basic-server-information)
- [Setup IP binding on each server](#setup-ip-binding-on-each-server)
- [Install HAProxy](#install-haproxy)
- [Configure HAProxy](#configure-haproxy)
- [Install keepalived](#install-keepalived)
- [Configure keepalived](#configure-keepalived)
- [Start keepalived automatically](#start-keepalived-automatically)
- [Sources](#sources)

## Background
**keepalived** is useful for setting up active-standby failover for services. When an outage is detected with the monitored service on the **MASTER** server, **keepalived** starts that service up on the **BACKUP** server. All communication between the **MASTER** and **BACKUP** servers takes place over a virtual IP (**VIP**). In this setup, **keepalived** monitors the status of **HAProxy**, a load balancer and reverse proxy service.

## Basic Server Information
A minimum of two servers is necessary for keepalived to work. One server should be designated **MASTER**, the other **BACKUP**. Communication between servers requires a **VIP**.

haproxy01.domain.com - **10.10.10.11** (**MASTER**)<br>
haproxy02.domain.com - **10.10.10.12** (**BACKUP**)<br>
Shared Virtual IP - **10.10.10.99** (**VIP**)<br>

## Setup IP binding on each server
**keepalived** uses a **VIP** to communicate service status between servers. For Ubuntu, the ability to bind IPs not defined in the interfaces file needs to be enabled.

- Perform the following steps on both servers.

1. Open the sysctl.conf file.<br>
`sudo vi /etc/sysctl.conf`
2. Add the following line at the end of the file.<br>
`net.ipv4.ip_nonlocal_bind = 1`
3. Make the new setting take effect.<br>
`sudo sysctl -p`

## Install HAProxy
The **HAProxy** version available from the Ubuntu repos is always out of date. Instead, the latest stable version of **HAProxy** can be obtained from a custom repo. The following site is useful for determining specfic installation steps for Debian distros: https://haproxy.debian.net/

The installation steps specific to Ubuntu 16.04 are included below.

- Perform the following steps on both servers.

1. Add the repo and dependencies
```
sudo apt install software-properties-common
sudo add-apt-repository ppa:vbernat/haproxy-1.7
```
2. Download and install haproxy
```
sudo apt update
sudo apt install haproxy
```

## Configure HAProxy
- HAProxy configuration is outside the scope of these instructions.
- Configure HAProxy on MASTER, and copy that configuration over to BACKUP. The HAProxy configuration file should be the same on both servers.

## Install keepalived
The keepalived version available from the Ubuntu repos is always out of date, and has some significant bugs. Instead, the latest stable version of keepalived can be obtained by building the installation package from source files. 

- Perform the following steps on both servers.

1. `sudo apt update`
2. `sudo apt install build-essential libssl-dev`
3. Go to http://www.keepalived.org/download.html and copy the link to the latest keepalived tarball. Use wget to download the file. Example: `wget http://www.keepalived.org/software/keepalived-1.3.9.tar.gz`
4. `sudo tar xzvf keepalived*`
5. `cd keepalived*`
6. `sudo ./configure`
7. `sudo make`
8. `sudo make install`

## Configure keepalived
- Create a keepalived config file for the MASTER server
`sudo vi /etc/keepalived/keepalived.conf`

Some notes on things to change
ipsec_ah

## Start keepalived automatically
A lot of the insructions for setting up keepalived on Ubuntu deal with 14.04 or earler, which utilizes Upstart scripts. I didn't have much luck attempting to use them, but this init script did the trick.

1. Open `sudo nano /etc/init.d/keepalived`
2. Paste the contents from https://gist.github.com/JamieCressey/f5e011cb838b9d867834 into the `keepalived` file.

sudo chmod +x /etc/init.d/keepalived
sudo /etc/init.d/keepalived start
sudo update-rc.d keepalived defaults
`Might need to run again`
sudo /etc/init.d/keepalived start
sudo service keepalived status

## Sources:
- https://www.digitalocean.com/community/tutorials/how-to-set-up-highly-available-haproxy-servers-with-keepalived-and-floating-ips-on-ubuntu-14-04
- https://askubuntu.com/questions/623053/installing-keepalived-lastest-version
- https://www.digitalocean.com/community/questions/keepalived-for-ubuntu-16
- https://gist.github.com/JamieCressey/f5e011cb838b9d867834

## Extras
Configure G-Suite SMTP relay for keepalived
https://support.google.com/a/answer/2956491
Use stmp-relay.gmail.com for email server
