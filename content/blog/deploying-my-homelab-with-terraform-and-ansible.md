---
author: Liam
title: "Deploying and configuring my homelab with Ansible and Terraform"
date: 2023-06-18
description: "A review of how I've used Ansible and Terraform to deploy and configure my homelab"
tags: ["homelab", "ansible", "configuration", "iac", "automation", "deployment", "provisioning", "infrastructure"]
thumbnail: img/server-cartoon.jpeg
---

## Introduction

One of the main projects for my homelab was to be able to deploy and configure it using Terraform and Ansible. I wanted to be able to automate the process of provisioning my homelab to remove as much of the manual work as possible. If you've not read my [Introduction to Terraform]({{< ref "introduction-to-terraform" >}}) and [Introduction to Ansible]({{< ref "introduction-to-ansible" >}}) posts I recommend taking a look at those to give you an introduction to what these tools are, how they can help you and their benefits.

After many years of repeating the same task of setting up servers, installing and configuring the software, my homelab provided the perfect project to grasp the concepts and gain some experience using Terraform and Ansible to provision and configure systems and software to learn about Infrastructure as Code and configuration management.

## Getting Started

Having never used Ansible or Terraform before I firstly started doing a lot of Googling to learn some basic usage of both tools. This led to others blog posts and some YouTube videos notably;

* [Orchestrate Your Systems with Ansible Playbooks - How to Home Lab Part 10](https://www.dlford.io/ansible-orchestration-how-to-home-lab-part-10/) - Part of a series of useful and interesting blog posts on homelabbing from Dan Ford's blog
* [Using Terraform and Cloud-Init to deploy and automatically monitor Proxmox instances](https://yetiops.net/posts/proxmox-terraform-cloudinit-saltstack-prometheus/#creating-a-template) - A blog post by yetiops.net on managing and monitoring instances in Proxmox
* [Ansible 101 by Jeff Geerling](https://www.jeffgeerling.com/blog/2020/ansible-101-jeff-geerling-youtube-streaming-series) - I highly recommend as Jeff Geerling provides some in-depth and easy to understand tutorials
* [Automating my Homelab with Ansible - Jeff Geerling](https://www.jeffgeerling.com/blog/2022/automating-my-homelab-ansible-ansiblefest-2022) - Jeff talks about how he automates various components with Ansible
* [Ansible Examples of Playbooks](https://github.com/ansible/ansible-examples) - Starter examples of Ansible Playbooks from Ansible!
* [Automate your Docker deployments with Ansible](https://youtu.be/CQk9AOPh5pw) - Christian Lempa's video on automating Docker deployments with Ansible
* [Simple automation for all your Linux servers with Ansible](https://youtu.be/uR1_hlHxvhc) - Christian Lempa gives and introduction and tutorial on using Ansible automations
* [Automate your virtual lab environment with Ansible and Vagrant](https://youtu.be/7Di0twyxw1M) - Another Christian Lempa tutorial on using Ansible to provision VMs with Vagrant

If you're not familiar with [Jeff Geerling](https://www.youtube.com/@JeffGeerling), [Christian Lempa](https://www.youtube.com/@christianlempa) and [Techo Tim](https://www.youtube.com/@TechnoTim) I highly recommend heading over to their YouTube channels as they put out some great content related to homelab and tech in general. Furthermore, I heavily relied on the [Ansible Documentation](https://yetiops.net/posts/proxmox-terraform-cloudinit-saltstack-prometheus/#creating-a-template) and [Terraform Provider Docs](https://registry.terraform.io/providers/Telmate/proxmox/latest/docs) for the `telmate/proxmox` provider in this case but any provider you want to use will likley have examples in the docs. There are enough examples and the documentation is easy to follow that I didn't find it too hard to get things working.

Using the above resources I started hacking away at Terraform to firstly provision a dev VM in Proxmox from a template. Once I got that working I then spent _many hours_ building and getting my Ansible playbooks to work against my test dev VM. Once I was happy with the result and that I could go from nothing to a VM provisioned by Terraform and then configured using Ansible, I set up the relevant playbook, roles and additional files for machines I wanted. I discussed what machines and services I run [My Homelab Setup Intro]({{< ref "my-homelab-setup" >}}), but the main goal was to go from nothing to running my media services in one VM and some other Docker containers in another VM, plusd other services as my homelab evolves.

## Provisioning and Configuring the homelab

After _many, many_ iterations of the Terraform and Ansible code I was finally able to go from nothing to a running homelab within minutes. Setting up these VMs manually, installing the software, configuring them etc would usually take hours of work, a whole evening or more :hourglass_flowing_sand:. Now I can sit back and let Terraform and Ansible take care of the lifting :relaxed:. If you're interested in the incremental progress I made to do this, you can find [my Homelab GitHub repo here](https://github.com/liamtill/homelab) where you can browse through the `ansible` and `terraform` directories to see how I have set things up to provision my machines. At the moment these are a work in progress and are a little rough round the edges. There is plenty of room for improvement which I'll be doing over the next few weeks/months and will write this up on the blog. What might also be useful is to go through the commit history, that way you can see how over time I built up the Terraform and Ansible code to get tot eh stage of being able to deploy my homelab VMs and services.

From the beginning I tried to make use of _roles_, _variables_, _vault_ for secrets and of course format my _playbooks_ in a sequence that made sense to me and followed general best practices and structure. You'll notice there are other playbooks in my GitHub repo for MySQL, PostgreSQL and some initial monitoring. There are still some manual steps in here of course, from the initial Proxmox setup, creating the VM template that Terraform will use and then actually running each of the Terraform plans and getting the IP of a machine so that I can put it in my Ansible inventory.

### Example Terraform plan for my docker VM

Below is an example of the Terraform plan I use to deploy a docker VM in Proxmox. In the plan below, you can see the basic VM settings and this takes about a minute to provision :smiley:, much quicker than clicking around. Once complete, I have a VM based on an Ubuntu Cloud 20.04 image that has been pre-configured using cloud-init to set my username, password and SSH keys so I can immediatly connect to the VM.

{{< accordion id="accordion-default" >}}
  {{< accordion-item header="Docker VM Terraform Plan">}}
    ```terraform
    resource "proxmox_vm_qemu" "docker" {

    # pve node
    target_node = "nimbus-pve"

    # define machine name
    name = "docker"
    desc = "Docker VM"
    
    # enable the qemu-guest-agent
    agent = 1

    # vm template to clone from
    clone = "ubuntu-cloud-20.04"
    # full clone instead of linked clone
    full_clone  = "true"

    # vm settings
    os_type = "cloud-init"
    cores = 2
    sockets = 1
    cpu = "host"
    memory = 2048
    scsihw = "virtio-scsi-pci"
    bootdisk = "scsi0"
    boot = "cdn"
    onboot = "true"

    # define network hw
    network {
      bridge = "vmbr0"
      model = "virtio"
    }

    # define storage
    disk {
        storage = "local-lvm"
        type = "scsi"
        size = "64G"
        # enable ssd emulation
        ssd = 1
    }


    lifecycle {
    ignore_changes  = [
      network,
    ]
  }

  provisioner "local-exec" {
    command = "echo ${self.default_ipv4_address}"
  }

}
  {{< /accordion-item >}}
{{< /accordion >}}

One alternative to this is something called "Golden Images". This is where you might create a "Golden" VM image which already has a bunch of software and configuration done, for example having Docker or nginx or some other service installed. However, I like the approach of using a bare image that I can clone many times and then use Ansible to install and configure the software I require.

### Example of Ansible playbook for my docker VM

Below is the Ansible playbook that I run against the docker VM provisioned with the Terraform plan above. I've used used _variables_, _roles_ and _templating_ where possible to make the playbook flexible.

{{< accordion id="accordion-default" >}}
  {{< accordion-item header="Ansible Playbook for Docker VM">}}
    ```yaml
    - hosts: docker-vm
  roles:
    - base
    - user
    - docker
    - deploy_watchtower
    - rclone
  tasks:

    # install pymysql dependency on host
    - name: install python3 pymysql 
      become: yes
      apt:
        name: python3-pymysql

    # need to make sure this proxy network exists first, before running compose files
    - name: Create proxy network for traefik and containers
      community.docker.docker_network:
        name: proxy

    # setup bookstack
    - name: setup bookstack
      import_tasks: setup-bookstack.yaml

    - name: creating required directories
      file:
        path: "/home/{{ ansible_user }}/docker/{{ item }}"
        state: directory
      loop:
          - syncthing
          - syncthing/data
          - portainer
          - portainer/data
          - adminer
          - pgadmin
          - paperless-ng
          - paperless-ng/data
          - paperless-ng/media
          - paperless-ng/export
          - paperless-ng/consume

    - name: copy docker compose files
      template:
        src: "templates/docker-compose/{{ item.file }}"
        dest: "/home/{{ ansible_user }}/docker/{{ item.name }}/docker-compose.yaml"
      loop:
        - { name: "portainer", file: "portainer.j2" }
        - { name: "adminer", file: "adminer.j2" }
        - { name: "pgadmin", file: "pgadmin.j2" }
        - { name: "syncthing", file: "syncthing.j2" }
        - { name: "paperless-ng", file: "paperless-ng.j2" }

    - name: copy env files
      copy:
        src: "templates/env/{{ item.file }}"
        dest: "/home/{{ ansible_user }}/docker/{{ item.name }}/.env"
      loop:
         - { name: "syncthing", file: "syncthing.env" }

    - name: copy pgadmin env file
      template:
        src: templates/env/pgadmin.j2
        dest: /home/{{ ansible_user }}/docker/pgadmin/.env

    - name: copy paperless-ngx env file
      template:
        src: templates/env/paperless-ng.j2
        dest: /home/{{ ansible_user }}/docker/paperless-ng/.env

    - name: setup traefik
      import_tasks: setup-traefik.yaml

    - name: run containers
      community.docker.docker_compose:
        project_src: "/home/{{ ansible_user }}/docker/{{ item }}"
        restarted: true
        files:
          - docker-compose.yaml
      loop:
        - portainer
        - adminer
        - pgadmin
        - syncthing
        - paperless-ng
      register: output

    - ansible.builtin.debug:
        var: output
  {{< /accordion-item >}}
{{< /accordion >}}

Head over to my [GitHub](https://github.com/liamtill/homelab/tree/main) to see more examples of my playbooks. You'll also find many examples of playbooks on the GitHubs os Jeff Geerling, TechnoTim and Christian Lempa which are very useful.

## Future Plans

As hinted above, there are improvements that can be made, and with many things, learning is an iterative process;

* Merge Terraform states and potentially plans instead of individual folders
* Use Terraform modules to improve structure and reduce duplication - this I believe is a best practice I should follow
* Have Terraform grab the IP of the provisioned machine and either run the Ansible playbook or dynamically update the inventory
* Have a CI/CD system manage the changes so that when I push to my repo a CI/CD flow will trigger to apply any infrastructure changes from Terraform and configuartion from Ansible

## Conclusion

Hopefully from this post you can see the benefits of using tools such as Terraform and Ansible. Even in the homelab environment they are incredibly useful and the homelab provides the perfect playground for learning such tools widely used in industry. Within _not so much time_ over evenings I now have a working deployment and configuration of my homelab. Therefore, if anything ever goes wrong or I simply want to redeploy all I need to do is get my GitHub repo, set up the relevant credentials for Terraform and to interact with Proxmox and then I can run all my plans and playbooks to setup my homelab.
