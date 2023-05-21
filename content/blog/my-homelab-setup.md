---
author: Liam
title: "My Homelab Setup"
date: 2023-05-21
description: "Homelab hardware and services setup"
tags: ["homelab", "system", "services", "hardware", "proxmox", "docker", "raspberry pi"]
thumbnail: /img/lab_photos/Homelab_Featured.jpg
---


## Intro

I've always had some sort of homelab over the years, which had consisted solely of Raspberry Pi's (the ones shown in the photos and still partly in use now). I'd always liked using these as they are great little low powered devices that are fairly capable. I've had the same 3 Pi's, one v2 model B and two v3 model B's for many years. I've considered selling them and getting one NUC or similar but I've become somewhat attached to them at this point as they've served me well over the years. I've ran various things on them, such as

- WRF - The Weather and Research Forecast Model. As a project whilst I was studying at the University of Oklahoma I built and ran a daily weather forecast model on the v2 Pi alone!
- weewx - for my weather station to collect data and send it to various websites
- PiAware - for tracking aircraft via ADS-B using an RTL-SDR
- HomeAssistant - to run various smart home devices like cameras, lightbulbs, automated routines, security alarm, media players etc
- Custom python code for smart home projects
- Stock price database
- plus a few other things over the years

## Current Hardware

### Main Server

<img src="/img/lab_photos/proxmox-server.jpg" title="Proxmox Server Case" width=300  class="rounded" align="right"/>

I've recently upgraded my homelab from the Pi's and my old laptop after wanting to run more services and learn some new skills. I planned on getting a dedicated NAS and a separate server (TinyMiniMicro) to run everything and be energy/space efficient (as I live in a small apartment). I ended up using the TinyMiniMicro Dell OptiPlex 3020 as my desktop. In the end I managed to get a MicroATX system on eBay which included everything for me to get started, and decided to do an "all-in-one" and to virtualise the NAS software. However, I found my NAS HDD would not fit in the small cube case and the original plan to have a small, quiet and efficient cube system went out the window and I now have a Fractal Define Mini C case. This case is great as it can fit multiple drives and is very quite due to the sound proofing. Now I have a quiet, efficient but not so small homelab 🤦‍♂️.

I run [Proxmox](https://www.proxmox.com/en/) on this machine (`nimbus`) as my hypervisor and base OS so I can run many virtual machines and containers. The specs of the machine are:

- Gigabyte GA-B150M-D3H-DDR3 Motherboard
- i5-6500 4 cores @ 3.20GHz and 32GB RAM
- 500GB SSD for host
- 10TB 3.5" Seagate IronWolf NAS HDD for media
- External 1TB 2.5" Toshiba drive for backups

<div class="clearfix">
    <img src="/img/lab_photos/proxmox-server-inside.jpg" title="Proxmox Server Inside Case" width=450 class="rounded float-start" />
    <img src="/img/lab_photos/proxmox-server-mobo1.jpg" title="Proxmox Server Motherboard" width=350 class="rounded float-end" />
</div>

### Raspberry Pi's

{{< image src="/img/lab_photos/rpis.jpg" ratio="21x9" class="rounded" >}}

These are the OG Pi's that are now mounted on my pegboard that used to be a stacked cluster. Only 1 (left; `cumulus`) is in use at the moment and runs [Tailscale](https://tailscale.com/) so I can VPN back to my network. These Pi's were running some docker containers, which are now moved to a VM in Proxmox. I plan to run monitoring on the middle Pi (`stratus`) using [UptimeKuma](https://github.com/louislam/uptime-kuma) and the 3rd Pi (right; `cirrus`) is looking for a project 😉 at the moment. 

### Janky laptop 💻  `#labgore`

<img src="/img/lab_photos/laptop-switch-pegboard.jpg" width=300 title="Janky laptop and Netgear Switch" class="rounded float-end" />

This horrific dismantling of my old laptop (`tropo`) has had a long life and was used for my PhD. It was a rather clunky case and I decided to get a new smaller laptop. This meant that I needed to find a use for it! 

I didn't want to have it sitting around in the case and I didn't want to get rid of it as it's now sentimental after everything we've been through. So I took it apart to use the motherboard only and was hosting docker containers and media services before I got my main machine (`nimbus`) and the 10TB drive. I planned on using the screen as a Grafana dashboard but never did so re-mounted the board to an acryllic sheet on another pegboard under my desk.

Laptop specs

- Motherboard from a HP Pavilion 15
- 120GB SSD and 8GB RAM
- AMD A10-5745M APU 4 cores @ 2.10GHz
- Noisy fan 📢

This pegboard also has a NETGEAR GS305 switch and a TP-Link TL-WR841N wireless router (aerials removed) that is also being used as a switch because I had it lying around. All traffic goes through these and then to a TP-Link PowerLine adapter linked to the main router in my apartment.

### What's going on with this naming

☁ Being a Meteorologist my servers have ended up being named after clouds or meteorology related terms ⛈

## Software / Services

{{< image src="/img/lab_photos/proxmox.jpg" title="Proxmox Dasboard" class="rounded">}}

The screenshot above shows my current work in progress. I run

- [OpenMediaVault (OMV)](https://www.openmediavault.org/) as a VM to be my virtual NAS for the 10TB drive
- PiHole as an LXC for DNS and ad-blocking
- Postgres database LXC
- MySQL Database LXC
- Custom python script for downloading stock data
- [Proxmox Backup Server (PBS)](https://www.proxmox.com/en/proxmox-backup-server) as a VM to take backups to the external drive
- docker - a VM to run docker containers
    * Traefik - reverse proxy serving local network with SSL
    * BookStack - Wiki docs
    * paperless-ngx - to digitise documents and use optical character recognition
    * Adminer - MySQL database management
    * PgAdmin - PostgreSQL database management
    * Portainer - Docker container management
    * Syncthing - sync client/server
    * Watchtower - MOnitor docker containers for updates
- media - dedicated VM for media services (*arrs)
    * Jellyfin - media server/player
    * Radarr - Movie collection manager
    * Sonarr - Series collection manager
    * nzbget - `nzb` downloader
    * Readarr - 2 instances for audiobooks and ebooks
    * Prowlarr - Usenet indexer manager
- totoscrafts - My wife's craft blog
- [Tailscale](https://tailscale.com/) for VPN
- All of my OSes are Ubuntu Server 20.04 or 22.04.

There are also some machine templates I use to create VMs and containers, plus some unused VM's which I'll be coming back to in the future to learn Kubernetes 😅.

## Why have a homelab?

If you're unsure what the point is of having a homelab, heres my input. I find it very useful as it allows me to host my own services, which many people enjoy doing and you can head over to [r/selfhosted](https://www.reddit.com/r/selfhosted/) and [r/homelab](https://www.reddit.com/r/homelab/) for more info. This can range from media players/services to many different docker containers for all different types of software/services from monitoring, finance management, project management, software development and coding, web servers, databases, your own cloud drive, ad blocking, wiki documentation, smart home software... the list goes on and on. Take a look at [Docker Hub](https://hub.docker.com) and [linuxserver.io](https://docs.linuxserver.io/) for more info on the docker containers you can easily run on almost any machine.

Apart from being able to run a million docker containers and many applications to serve almost any need you have, homelabs are great learning environments. They are fantastic to learn/develop new skills your interested in, to further your career skillset or to replicate enterprise services you use daily. They provide a platform where you can mess with things at your own will. If it goes wrong you can just delete it all and start again. Homelabs are very useful developement and learning environments. 

If you want more info, take a look at [r/homelab](https://www.reddit.com/r/homelab). Be careful though, you may end up spending more money than you want to on servers and TB's of storage!

## Current and Future Plans

My current plans are to

- Run and maintain my current services using automation and add services
- Learn Ansible and Terraform to provision and configure my homelab, so I can do as minimal manual setup and maintenance as possible

My future plans are to

- Learn Kubernetes to run my homelab and for a personal project
- Learn CI/CD along with Kubernetes so I can go from development to integration and deployment of a personal project application
- Keep blogging about my progress and skills I learn

My ultimate goals are to 

- Be able to deploy my homelab into a Kubernetes cluster with minimal manual input and have the lab fully managed via CI/CD flow
- Be able to deploy an application I've developed into a Kubernetes cluster and use CI/CD
- Continue to learn new skills to understand how applications go from code to fully highly available in production
- Improve my software development skillset and learning to write better code