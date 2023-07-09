---
author: Liam
title: "Configuring my laptop using Ansible"
date: 2023-07-09
description: "Configuring and setting up my laptop using Ansible from scratch"
tags: ["ansible", "configuration", "automation", "provisioning"]
thumbnail: img/laptop-configure.png
---

## Introduction

Every now and then I like to do a fresh install of my OS, which is currently Kubuntu, just to start fresh and clear out any junk I've installed over time. I'd also recently upgraded Kubuntu which had not gone so smoothly and some things were not quite working. So this was a good excuse to do a fresh install and leverage Ansible to configure my laptop. This by no means is revolutionary but is a good example for getting used to using Ansible.

The benefit of using Ansible means that whenever I do this again on any laptop I can simply clone my git repository and run the same playbook and get the same results, remember Ansible is idempotent. This also saves me the chore of installing all the software and if I want fine tuning any settings and customisation

Automating repetitive tasks or any tasks that you want the same outcome to happen each time, for example installing and configuring a laptop, is something I'm trying to move towards each time I come up against tasks that I've done a thousand times and never bothered to automate before. In this post I'll use a simple Ansible playbook to install software I want on my laptop.

## Kick starter script

You might be thinking how do I use Ansible to configure my laptop when it's not even installed and I have to run some `apt install` commands manually! This is where a kick starter or a "pre" script comes in, at least that's what I'm calling it. From a fresh OS install, all I need to do is clone [my `ansible_laptop` git repository](https://github.com/liamtill/ansible_laptop) and run `start.sh`:

```shell
git clone https://github.com/liamtill/ansible_laptop.git
./start.sh
```

The contents of my starter script are:

{{< accordion id="accordion-default" >}}
  {{< accordion-item header="bash kickstarter script">}}
    ```bash
    #!/bin/bash

    # Initial script to install requirements to run Ansible
    echo "Installing curl"
    sudo apt install -y curl

    echo "Installing pip and Ansible"
    curl https://bootstrap.pypa.io/get-pip.py -o /tmp/get-pip.py
    python3 /tmp/get-pip.py --user

    sudo apt install -y software-properties-common
    sudo add-apt-repository --yes --update ppa:ansible/ansible
    sudo apt install -y ansible
    echo "Checking Ansible version"
    ansible --version

    echo "Running playbook"
    # ensure ansible knows the sudo password
    ansible-playbook playbooks/main.yaml --ask-become-pass

    echo "Final Steps:"
    echo "Configure syncthing"
    echo "Configure ohmyzsh"
    echo "Log into services"
  {{< /accordion-item >}}
{{< /accordion >}}

This script

1. Makes sure `curl` is installed
2. Installs pip
3. Installs a dependency and adds the Ansible ppa to `apt`
4. Installs Ansible and then runs the main playbook
5. Prints out some reminder messages for final things I need to do, which could potentially be done using Ansible and dotfiles

You can add any initial steps you might require to your starter script that might be needed to run things in your playbook. Equally you may even just use a bash script to configure/provision your laptop and not need to use Ansible at all.

## Ansible playbook

I firstly define a basic `inventory.yml`, that uses the host `localhost` and the `local` connection:

```yaml
all:
  vars:
    ansible_user: liam
  hosts:
  children:
    localhost:
      ansible_connection: local
```

I add any tasks I want to run to `playbooks/main.yaml` and by separating playbooks and roles it gives me flexibility to build on this over time. In my git repository I have used some roles; one created by me to install docker and another called [`gantsign.oh-my-zsh`](https://github.com/gantsign/ansible-role-oh-my-zsh) to install and configure oh-my-zsh.

My tasks simply install a bunch of packages using apt, install some applications from deb and ensures the required configuration is done. At the time of this blog post my playbook is:

{{< accordion id="accordion-default" >}}
  {{< accordion-item header="Ansible Playbook to provision laptop">}}
    ```yaml
    ---
    - hosts: localhost
    become: yes
    vars:
        obsidian_version: "1.3.5"
        slack_version: "4.32.127"
        go_version: "1.20.5"

    tasks:
        - name: update and upgrade apt packages
          apt:
            upgrade: yes
            update-cache: yes

        - name: "Get lsb_release"
          shell: lsb_release -cs
          register: lsb_release
        - name: "Print lsb_release"
          debug:
            msg: "{{ lsb_release.stdout }}"

        - name: confirm keyrings dir exists for syncthing
          file:
            path: /usr/share/keyrings
            state: directory
        - name: add syncthing gpg key
          apt_key:
            url: https://syncthing.net/release-key.gpg
            keyring: /usr/share/keyrings/syncthing-archive-keyring.gpg
            state: present
        - name: add stable syncthing channel to sources
          apt_repository:
            repo: "deb [signed-by=/usr/share/keyrings/syncthing-archive-keyring.gpg] https://apt.syncthing.net/ syncthing stable"
            state: present

        - name: install gpg key for sublime
          apt_key:
            url: https://download.sublimetext.com/sublimehq-pub.gpg
            keyring: /etc/apt/trusted.gpg.d/sublimehq-archive.gpg
            state: present
        - name: add stable sublime channel to sources
          apt_repository:
            repo: "deb https://download.sublimetext.com/ apt/stable/"
            state: present

        - name: install gpg for terraform
          apt_key:
            url: https://apt.releases.hashicorp.com/gpg
            keyring: /usr/share/keyrings/hashicorp-archive-keyring.gpg
            state: present
        - name: add terraform main channel to sources 
          apt_repository:
            repo: "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com {{ lsb_release.stdout }} main"
            state: present
        
        - name: Add keepassxc repo
          apt_repository:
            repo: ppa:phoerious/keepassxc

        - name: apt update
          become: yes
          apt:
            update_cache: yes

        - name: Install packages
          apt:
            pkg:
            - gnome-keyring
            - apt-transport-https
            - sublime-text
            - syncthing
            - zsh
            - terraform
            - htop
            - tmux
            - keepassxc
            - npm

        - name: update npm
          shell: npm install -g npm

        - name: install nodejs using nodesource
          become: true
          shell: curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash

        - name: install nodejs
          apt:
            pkg: nodejs

        - name: install Obsidian from deb
          apt:
            deb: "https://github.com/obsidianmd/obsidian-releases/releases/download/v{{ obsidian_version }}/obsidian_{{ obsidian_version }}_amd64.deb"

        - name: install VS Code from deb
          apt:
            deb: https://az764295.vo.msecnd.net/stable/695af097c7bd098fbf017ce3ac85e09bbc5dda06/code_1.79.2-1686734195_amd64.deb

        - name: install mega.io cloud drive app from deb
          apt:
            deb: https://mega.nz/linux/repo/xUbuntu_22.04/amd64/megasync-xUbuntu_22.04_amd64.deb

        - name: install Chrome from deb
          apt:
            deb: https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb

        - name: install Slack from deb
          apt:
            deb: "https://downloads.slack-edge.com/releases/linux/{{ slack_version }}/prod/x64/slack-desktop-{{ slack_version }}-amd64.deb"

        - name: install docker using role
          include_role:
            name: docker

        - name: install and configure ohmyzsh
          include_role:
            name: gantsign.oh-my-zsh
          vars:
            users:
              - username: {{ ansible_user }}

        - name: Clone powerlevel10k zsh theme repo
          git:
            repo: https://github.com/romkatv/powerlevel10k.git
            dest: "/home/{{ ansible_user }}/powerlevel10k"
            depth: 1

        - name: add powerlevel10k theme to zsh config
          lineinfile:
            line: "source /home/{{ ansible_user }}/powerlevel10k/powerlevel10k.zsh-theme"
            path: "/home/{{ ansible_user }}/.zshrc"

        - name: ensure zsh is default shell
          become: yes
          shell: chsh -s $(which zsh)
        
        - name: start syncthing
          shell: /usr/bin/syncthing serve --no-browser --logfile=default &

        - name: download go tar
          get_url:
            url: "https://go.dev/dl/go{{ go_version }}.linux-amd64.tar.gz "
            dest: "/tmp/go{{ go_version }}.linux-amd64.tar.gz"

        - name: remove any existing go path
          file:
            path: /usr/local/go
            state: absent

        - name: untar go 
          unarchive:
            src: /tmp/go{{ go_version }}.linux-amd64.tar.gz
            dest: /usr/local

        - name: add go bin path to path
          shell: export PATH=$PATH:/usr/local/go/bin

        - name: add go path to .profile
          lineinfile:
            line: export PATH=$PATH:/usr/local/go/bin
            path: "/home/{{ ansible_user }}/.profile"

        - name: source .profile
          shell: ". /home/{{ ansible_user }}/.profile"

        - name: Install packages with snap
          snap:
            name: 
            - tradingview
            - hugo

  {{< /accordion-item >}}
{{< /accordion >}}

This fairly basic and can be can be expanded to also configure other things such as setting a wallpaper, user icon or deploying your custom dotfiles.

Dotfile deployment is another usage for a git repository and a shell script or Ansible as this allows you to easily deploy your custom dotfiles such as those for git, vim, tmux, bashrc or zshrc plus many others.

## Conclusion

In this post we saw another, yet simple, application for Ansible to configure my laptop. As we've done with servers we can use Ansible to configure any system whether it is your own laptop or 1000 user laptops in an organisation. Think of all those custom settings you have applied to your laptop and all the applications you have installed, now imagine having to set up your laptop exactly how you want it again and how long it will take and if you'll remember all those steps you did and to edit configuration files again.

Let automation tools do this for you!

## Resources

- [My Git repository for this post](https://github.com/liamtill/ansible_laptop)
