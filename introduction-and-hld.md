# A journey into containerizing my Home IoT network

## Table of Contents <!-- omit in toc -->
- [A journey into containerizing my Home IoT network](#a-journey-into-containerizing-my-home-iot-network)
  - [The idea](#the-idea)
  - [How to keep everything running?](#how-to-keep-everything-running)
  - [What I want to migrate](#what-i-want-to-migrate)
  - [High Level design](#high-level-design)
  - [A lot of work hey! Where do you start?](#a-lot-of-work-hey-where-do-you-start)

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

