# keepalived
Collected instructions for setting up keepalived with HAProxy on Ubuntu 16.04.

## Table of Contents
- [Background](#background)
- [Basic Server Information](#basic-server-information)
- [Setup IP binding on each server](#setup-ip-binding-on-each-server)
- [Install and configure HAProxy](#install-and-configure-haproxy)
- [Install keepalived](#install-keepalived)
- [Configure keepalived](#configure-keepalived)
- [Start keepalived automatically](#start-keepalived-automatically)
- [Sources](#sources)

## Background
**keepalived** is useful for setting up active-standby failover for services. When an outage is detected with the monitored service on the **MASTER** server, **keepalived** starts that service up on the **BACKUP** server. Both the **MASTER** and **BACKUP** servers share a virtual IP (**VIP**) which allows only one server to be active at a time. In this setup, **keepalived** monitors the status of **HAProxy**, a load balancer and reverse proxy service.

## Basic Server Information
A minimum of two servers is necessary for keepalived to work. One server should be designated **MASTER**, the other **BACKUP**. Both servers share a **VIP**.

haproxy01.domain.com - **10.10.10.11** (**MASTER**)<br>
haproxy02.domain.com - **10.10.10.12** (**BACKUP**)<br>
Shared Virtual IP - **10.10.10.99** (**VIP**)<br>

## Setup IP binding on each server
**keepalived** uses a **VIP** to designate the active server. For Ubuntu, the ability to bind IPs not defined in the interfaces file needs to be enabled for the **VIP** to work.

**Perform the following steps on both servers.**

1. Open the sysctl.conf file.<br>
`sudo vi /etc/sysctl.conf`
2. Add the following line at the end of the file.<br>
`net.ipv4.ip_nonlocal_bind = 1`
3. Make the new setting take effect.<br>
`sudo sysctl -p`

## Install and configure HAProxy
The **HAProxy** version available from the Ubuntu repos is always out of date. Instead, the latest stable version of **HAProxy** can be obtained from a custom repo. The following site is useful for determining specfic installation steps for Debian distros: https://haproxy.debian.net/. The installation steps specific to Ubuntu 16.04 are included below.

**Perform the following steps on both servers.**

1. Add the repo and dependencies.
```
sudo apt install software-properties-common
sudo add-apt-repository ppa:vbernat/haproxy-1.7
```
2. Download and install **HAProxy**.
```
sudo apt update
sudo apt install haproxy
```
3. Make any necessary edits to the **HAProxy** configuration on **MASTER**, and copy that configuration over to **BACKUP**. The configuration file should be the same on both servers.

## Install keepalived
The **keepalived** version available from the Ubuntu repos is always out of date, and has some significant bugs. Instead, the latest stable version of **keepalived** can be obtained by building the installation package from source files. 

**Perform the following steps on both servers.**

1. Update the package list.<br>
`sudo apt update`
2. Install `build-essential` and `libssl-dev`.<br>
`sudo apt install build-essential libssl-dev`
3. Change directories to `home`.<br>
`cd ~`
4. Go to http://www.keepalived.org/download.html and copy the link to the latest keepalived tarball. Use wget to download the file.<br>
`wget http://www.keepalived.org/software/keepalived-1.3.9.tar.gz`
5. Expand the keepalived tarball.<br>
`sudo tar xzvf keepalived*`
6. Change directories to the extracted directory.<br>
`cd keepalived*`
7. Build and install **keepalived**.
```
sudo ./configure
sudo make
sudo make install
```

## Configure keepalived
The config files for **keepalived** **MASTER** and **BACKUP** servers are included below with comments, and also as a downloadable files in this git repo. Items in `< >` need to be edited.

- `notification_email` - address the alerts will be sent to
- `notification_email_from` - address the alerts will be sent from
- `smtp_server` - SMTP address the alerts will be sent through
- `router_id` - designation for **keepalived** instance
- `enable_script_security` - prevents running scripts configured to run as root if any part of the path is writable by a non-root user
- `vrrp_script chk_haproxy` - script that checks status of **HAProxy** every 2 seconds
- `interface` - name of interface, use `ifconfig` to determine active adapter name
- `auth_type AH` - sets the authentication type for communication between servers to IPSEC-Authentication Header (AH), which is safer than the alternative plain-text (PASS)
- `auth_pass` - password for connunication between servers, only the first eight (8) characters are used
- `virtual_ip_address` - shared **VIP** for both servers
- `track_script` - which script **keepalived** monitors to decide if failover is necessary

- Create a **keepalived** config file for the **MASTER** server.<br>
`sudo vi /etc/keepalived/keepalived.conf`

```
global_defs {
  notification_email {
    <email@domain.com>
  }
  notification_email_from <haproxy01@domain.com>
  smtp_server <smtp.domain.com>
  smtp_connect_timeout 30
  router_id <haproxy01>
  enable_script_security
}
vrrp_script chk_haproxy {
  script "/usr/bin/killall -0 haproxy"
  interval 2
  weight 2
}
vrrp_instance haproxy {
  state MASTER
  interface <ens160>
  priority 101
  virtual_router_id 51
  smtp_alert
  authentication {
    auth_type AH
    auth_pass <12345678>
  }
  virtual_ipaddress {
    <10.10.10.99>
  }
  track_script {
    chk_haproxy
  }
}
```

- Create a **keepalived** config file for the **BACKUP** server.<br>
`sudo vi /etc/keepalived/keepalived.conf`

```
global_defs {
  notification_email {
    <email@domain.com>
  }
  notification_email_from <haproxy02@domain.com>
  smtp_server <smtp.domain.com>
  smtp_connect_timeout 30
  router_id <haproxy02>
  enable_script_security
}
vrrp_script chk_haproxy {
  script "/usr/bin/killall -0 haproxy"
  interval 2
  weight 2
}
vrrp_instance haproxy {
  state BACKUP
  interface <ens160>
  priority 100
  virtual_router_id 51
  smtp_alert
  authentication {
    auth_type AH
    auth_pass <12345678>
  }
  virtual_ipaddress {
    <10.10.10.99>
  }
  track_script {
    chk_haproxy
  }
}
```

## Start keepalived automatically
Many of the insructions for setting up **keepalived** on Ubuntu deal with 14.04 or earler, which utilizes Upstart scripts. None of them work with 16.04, but the init script below does.

1. Download the `keepalived` file from the repo to the location below.<br>
`/etc/init.d/`
2. Execute the following commands.
```
sudo chmod +x /etc/init.d/keepalived
sudo /etc/init.d/keepalived start
sudo update-rc.d keepalived defaults
```

## Sources:
- https://www.digitalocean.com/community/tutorials/how-to-set-up-highly-available-haproxy-servers-with-keepalived-and-floating-ips-on-ubuntu-14-04
- https://askubuntu.com/questions/623053/installing-keepalived-lastest-version
- https://www.digitalocean.com/community/questions/keepalived-for-ubuntu-16
- https://gist.github.com/JamieCressey/f5e011cb838b9d867834

## Extras
Configure G-Suite SMTP relay for keepalived
https://support.google.com/a/answer/2956491
Use stmp-relay.gmail.com for email server
