# keepalived
Collected instructions for setting up keepalived with HAProxy on Ubuntu 16.04

## Table of Contents
- [Background](#background)
- [Basic Server Information](#basic-server-information)
- [Setup IP binding on each server](#setup-ip-binding-on-each-server)
- [Install HAProxy](#install-haproxy)
- [HAProxy Configuration](#haproxy-configuration)
- [Install keepalived](#install-keepalived)
- [Start keepalived automatically](#start-keepalived-automatically)
- [Configure keepalived](#configure-keepalived)

## Background
Use the following instructions to configure an active-standby setup for HAProxy using keepalived. While these instructions are written for Ubuntu 16.04, they can be adapted for other Linux distros if needed.

## Basic Server Information
You'll need a minimum of two servers for keepalived to work. One server should be designated MASTER, the other BACKUP.

haproxy01.domain.com - 10.10.10.11 (MASTER)
haproxy02.domain.com - 10.10.10.12 (BACKUP)

## Setup IP binding on each server
keepalived needs a shared virtual IP (VIP) in order for communication between the two instances to be possible. First, you'll need to enable the ability to bind IPs not defined in the interfaces file.

- Open the sysctl.conf file
`sudo vi /etc/sysctl.conf`
- Add the following line at the end of the file
`net.ipv4.ip_nonlocal_bind = 1`
- Make the new setting take effect
`sudo sysctl -p`
- Perform these steps on both servers

## Install HAProxy
Use the following site to determine the installation steps specific to your Linux distro: https://haproxy.debian.net/

- Generally, the latest version is prefereable to the stable version that ships with Ubuntu.
- Add the repo and dependencies
```
sudo apt install software-properties-common
sudo add-apt-repository ppa:vbernat/haproxy-1.7
```
- Download and install haproxy
```
sudo apt update
sudo apt install haproxy
```

## HAProxy Configuration
Setup your HAProxy configuration on MASTER, and copy that configuration over to BACKUP. The HAProxy configuration file needs to be the same on both servers.

## Install keepalived
You could use `sudo apt install keepalived`, but the default version from Ubuntu's repos is outdated and posesses a few bugs. Instead, you'll need to build and install keepalived from the source files.

- Perform the following steps on both servers.

1. `sudo apt update`
2. `sudo apt install build-essential libssl-dev`
3. Go to http://www.keepalived.org/download.html and copy the link to the latest keepalived tarball. Use wget to download the file. Example: `wget http://www.keepalived.org/software/keepalived-1.3.9.tar.gz`
4. `sudo tar xzvf keepalived*`
5. `cd keepalived*`
6. `sudo ./configure`
7. `sudo make`
8. `sudo make install`

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

## Configure keepalived
- Create a keepalived config file for the MASTER server
`sudo vi /etc/keepalived/keepalived.conf`

Some notes on things to change
ipsec_ah

## Sources:
- https://www.digitalocean.com/community/tutorials/how-to-set-up-highly-available-haproxy-servers-with-keepalived-and-floating-ips-on-ubuntu-14-04
- https://askubuntu.com/questions/623053/installing-keepalived-lastest-version
- https://www.digitalocean.com/community/questions/keepalived-for-ubuntu-16
- https://gist.github.com/JamieCressey/f5e011cb838b9d867834


Internal Resolution
For internal resolution, create CNAME records pointing to the HAProxy VIP (192.168.1.99)

Configure SMTP Relay for internal apps
https://support.google.com/a/answer/2956491
Use stmp-relay.gmail.com for email server

Setup a VIP (192.168.1.99) and reservation
setup HAProxy1 (10.10.10.11) and reservation with MAC
setup HAProxy2 (10.10.10.12) and reservation with MAC
