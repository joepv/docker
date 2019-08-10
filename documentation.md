# A journey into containerizing my Home IoT network

## Table of Contents <!-- omit in toc -->
- [A journey into containerizing my Home IoT network](#a-journey-into-containerizing-my-home-iot-network)
  - [The idea](#the-idea)
  - [How to keep everything running?](#how-to-keep-everything-running)
  - [What I want to migrate](#what-i-want-to-migrate)
  - [High Level design](#high-level-design)
  - [A lot of work hey! Where do you start?](#a-lot-of-work-hey-where-do-you-start)
- [Install and harden Debian 10 to build a Docker host](#install-and-harden-debian-10-to-build-a-docker-host)
  - [Minimal server installation on Intel® NUC Kit](#minimal-server-installation-on-intel%c2%ae-nuc-kit)
  - [First install Vim!](#first-install-vim)
  - [Set a static IP address and correct DNS server](#set-a-static-ip-address-and-correct-dns-server)
  - [Granting my personal user administrative privileges](#granting-my-personal-user-administrative-privileges)
  - [Configure a basic firewall](#configure-a-basic-firewall)
  - [Set up SSH keys for easy and secure remote access](#set-up-ssh-keys-for-easy-and-secure-remote-access)
    - [Create the RSA key pair](#create-the-rsa-key-pair)
    - [Copy the public key to the server](#copy-the-public-key-to-the-server)
    - [Disable password authentication the server](#disable-password-authentication-the-server)
    - [Disable SSH login for the root user](#disable-ssh-login-for-the-root-user)

## The idea

In a series of blogs I write about containerizing my Home IoT network with Docker. At the moment my network runs on an Intel NUC with Windows 10 and a Debian 9 Linux machine in VirtualBox. This is not scalable and migrating all services to Docker containers I am ready to lift and shift my whole network to the cloud when I want to. And it's just fun to explore!

## How to keep everything running?

When exploring and learning Docker I want to keep my services running. Hey, I got lazy with all my automation's. I borrowed a temporary machine from my colleague and Docker enthusiast Rik Lempens to run my current platform on, so I can format c: my own machine and set it up from scratch.

## What I want to migrate

The following services are running on my network:

| Service          | What it does                     | Operating System |
| ---------------- | -------------------------------- | ---------------- |
| Node-RED         | For interconnecting IoT devices  | Debian 9         |
| MQTT broker      | For IoT devices communication    | Debian 9         |
| WordPress        | Personal website for family      | Debian 9         |
| MariaDB          | Database for WordPress           | Debian 9         |
| piHole           | Network-wide Ad Blocking         | Debian 9         |
| Veaam PN         | Simple and secure VPN networking | Ubuntu 18.04     |
| UniFi Controller | Network Management Controller    | Windows 10 Pro   |
| PowerBI gateway  | Power Monitoring Dashboard       | Windows 10 Pro   |

## High Level design

I chose Debian 10 as operating system running on the bare metal Intel NUC with the latest stable Docker release at installation date.

The UniFi controller and all Linux services and are going to be migrated to containers. To be able to run the Microsoft Power BI gateway I set up a headless Virtualization Server using KVM.

Besides the current services I'm building a container with custom code to read the data from my Nefit heat pump. Another container with SiriDB is created to save the time-series data. With SiriDB I'm going to analyse the heat pump data to tune my central heating/cooling.

After containerizing my whole IoT network looks like:

| Service          | What it does                       | Operating System                         |
| ---------------- | ---------------------------------  | ---------------------------------------- |
| Node-RED         | For interconnecting IoT devices    | Docker container on Debian 10            |
| MQTT broker      | For IoT devices communication      | Docker container on Debian 10            |
| SiriDB           | Time Series database for Heat Pump | Docker container on Debian 10            |
| WordPress        | Personal website for family        | Docker container on Debian 10            |
| MariaDB          | Database for WordPress             | Docker container on Debian 10            |
| piHole           | Network-wide Ad Blocking           | Docker container on Debian 10            |
| Veaam PN         | Simple and secure VPN networking   | Docker container on Debian 10            |
| UniFi Controller | Network Management Controller      | Docker container on Debian 10            |
| PowerBI gateway  | Power Monitoring Dashboard         | Microsoft Server 2019 on KVM with VirtIO |

## A lot of work hey! Where do you start?

The first step is to install a Windows 10 copy on my borrowed Intel NUC and copy/run the virtual machines on this computer. Then I'm going to install Docker and migrate every service one by one.

How I install Debian 10 with Docker, tune and harden the server I will write in my next article, so stay tuned!

# Install and harden Debian 10 to build a Docker host

In part 1 of my journey into containerizing my Home IoT network I sketched my IoT landscape and what it would be when it is all containerized. In this article I write about configuring the system for optimal use and securing (hardening) the default Debian 10 installation for remote access with SSH.

## Minimal server installation on Intel® NUC Kit

I assume you know how you install Debian. Remember to use a strong root and user password. I installed Debian 10 in text mode and only selected the packages `SSH server` and `standard system utilities` to get a minimal server system.

![minimal server install](img/debian-package-selection.png)

Note: I have an Intel® NUC Kit NUC6i5SYB, and you have to separately download the Wi-Fi drivers. As I connect my NUC with the LAN interface to the network I disabled Wi-Fi (and bluetooth) in the BIOS.

## First install Vim!

The system I installed is minimal. Even *Vim* is not installed! Therefore I login at the *console* with the *root* account and install Vim as the first step after installation: `apt-get install vim`.

## Set a static IP address and correct DNS server

Debian installs by default with DHCP enabled. Edit the file `/etc/network/interfaces` to change it to a static address:

```
auto eno1
iface eno1 inet static
    address 192.168.1.100/24
    gateway 192.168.1.254
```

To set the correct DNS servers edit `/etc/resolv.conf` and add your Internet router and Google's DNS servers as back-up:

```
nameserver 192.168.1.254
nameserver 8.8.8.8
```

Restart the system to make sure it boots correctly. Debian 10 has fully switched to *Systemd* which means that the traditional commands to reboot a system like `shutdown -r now` or `reboot` will not work anymore. The command to reboot your system is `systemctl reboot`.

## Granting my personal user administrative privileges

To avoid using the *root* account for administrative purposes I granted my login *joep* administrative privileges. This will allow my personal account to run commands with administrative privileges by using the `sudo` command. 

First install the *sudo* package (logged in as root): `apt-get install sudo` and add your account to the *sudo* group: `usermod -aG sudo joep`.

## Configure a basic firewall

Running a firewall on every server is a best practice. I use the UFW firewall to set firewall policies and manage exceptions. First I have to install the UFW package: `sudo apt-get install ufw`

I want the firewall to allows SSH connections so I can log in from a remote machine. I allow this by typing: `sudo ufw allow OpenSSH` and enable the firewall with the command `sudo ufw enable`. Check if SSH is still allowed by typing: `sudo ufw status`:

![ufw ssh status](img/debian-ufw-ssh-status.png)

## Set up SSH keys for easy and secure remote access

SSH keys are more difficult to hack than passwords and thus are more secure. Unlike passwords, the private SSH key isn't sent to the server and the SSH connection can only come from the client where the private key resides. For extra security you can add a password to your SSH key to enable multi-factor authentication to increase security even more.

### Create the RSA key pair

If you don't have a key pair yet, create one on your client machine with the command: `ssh-keygen`. By default a *2048-bit* RSA key pair is created.

![ssh-keygen](img/debian-ssh-keygen.png)

### Copy the public key to the server

The easiest way to copy your public key to your new Debian 10 server is to use the command: `ssh-copy-id joep@iotgateway`

```
Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'joep@iotgateway'"
and check to make sure that only the key(s) you wanted were added.
```

### Disable password authentication the server

Password-based authentication is still active, so your server is still exposed to brute-force attacks. Therefore I disable password-based authentication in the SSH daemon configuration by editing the file `/etc/ssh/sshd_config` and change the directive called *PasswordAuthentication*:

```
...
PasswordAuthentication no
...
```

### Disable SSH login for the root user

In the same configuration file I disable the *root* account for remote access to give my server some extra security.

```
...
PermitRootLogin no
...
```

Now restart the SSH daemon with the command: `sudo systemctl restart sshd.service`.


