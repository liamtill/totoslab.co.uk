---
author: Liam
title: "Introduction To Ansible - Automation | Configuration | Deployment"
date: 2023-05-29
description: "Introduction to using Ansible to help you configure, deploy, and manage your infrastructure and applications why using Ansible as a configuration management tool is useful."
tags: ["homelab", "ansible", "configuration", "iac", "automation", "software", "deployment"]
thumbnail: img/ansible-logo.png
---

## Introduction

In today's fast-paced world of software development, automation is key to ensuring that teams can deliver high-quality software quickly and efficiently. One of the most popular tools for IT automation is [Ansible](https://www.ansible.com/), an open-source platform that simplifies tasks such as configuration management, software deployment, and task automation.

## What is Ansible?

Ansible is an IT automation tool that helps you configure, deploy, and manage your infrastructure and applications. Ansible is fairly simple use, flexibile, and uses an agentless architecture which means that it doesn't require any additional software to be installed on the target machines, making it lightweight and easy to use. It uses a declarative language and tasks are defined in YAML (Yet Another Markup Language), so all you have to do is define the desired state of your system and Ansible takes care of making it happen.

## Key Benefits of Ansible

1. **Improved Efficiency**: Ansible's automation capabilities help reduce manual, repetitive tasks, freeing up you and your team to focus on more innovative work.
2. **Enhanced Collaboration**: Ansible's human-readable playbooks and modular design make it easy for team members to collaborate, share knowledge, and maintain consistency across projects.
3. **Reduced Errors**: By automating tasks and using version control for playbooks, Ansible helps minimize human error and ensures that your infrastructure is configured consistently and accurately.
4. **Faster Deployment**: Ansible's automation capabilities enable you to deploy applications and infrastructure updates more quickly, reducing downtime and improving time-to-market.
5. **Scalability**: Ansible's agentless architecture and flexible inventory management make it easy to scale your automation efforts as your organisation grows.
6. **Idempotency**: An operation is idempotent if the result of performing it once is exactly the same as the result of performing it repeatedly without any intervening actions. Ansible can _reduce configuration errors_ by applying the same action each time and the result should always be the same.

## Key Features of Ansible

1. **Agentless Architecture**: Unlike other automation tools that require agents to be installed on each managed node, Ansible relies on SSH (Secure Shell) or WinRM (Windows Remote Management) for communication. This eliminates the need for additional software on the target systems, reducing overhead and simplifying maintenance.
2. [Playbooks](https://docs.ansible.com/ansible/latest/playbook_guide/index.html): Ansible playbooks are YAML files that define the desired state of your infrastructure and the tasks required to achieve that state. Playbooks are easy to read, write, and share, making collaboration between team members seamless.
3. [Modules](https://docs.ansible.com/ansible/latest/module_plugin_guide/index.html): Ansible comes with a vast library of pre-built modules that cover a wide range of tasks, from managing files and services to interacting with APIs and cloud platforms. These modules make it easy to automate complex tasks without having to write custom scripts.
4. [Templating](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_templating.html): Ansible uses the Jinja2 templating engine, which allows you to create dynamic configurations based on variables and conditions. This makes it easy to manage complex configurations and ensure consistency across your infrastructure.
5. [Roles](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html): Ansible allows you to organise and group your automation tasks into reusable roles, which can be shared across multiple projects. This promotes reusability and helps you maintain a consistent and modular automation codebase. Roles can be created by users or downloaded from [Ansible Galaxy](https://galaxy.ansible.com/home).
6. [Inventory](https://docs.ansible.com/ansible/latest/inventory_guide/index.html): The inventory is a list of target machines that Ansible will manage. It can be defined in a simple text file or dynamically generated using scripts or other tools.
7. [Extensibility](https://docs.ansible.com/ansible/latest/dev_guide/index.html): Ansible is highly extensible, allowing you to create custom modules, plugins, and filters to meet your specific needs. This ensures that Ansible can grow with your organization and adapt to new technologies and requirements.

## Getting Started With Ansible

I followed the [Official Docs](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) to install Ansible using Python pip. Make sure you have `pip` installed (`curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py`) and for a _control node_ which is a system on which Ansible is installed where you run Ansible commands such as `ansible` or `ansible-inventory` on a control node you will need Python 3.9 - 3.11 for Ansible version 2.14 at the time of writing. From the official docs you can install ansible by: `python3 -m pip install --user ansible` to install Ansible for the current user. The docs [also have a basic getting started guide](https://docs.ansible.com/ansible/latest/getting_started/index.html) and the first two inventory examples below are from the official docs.

### Create an Inventory

Once installed, you'll need to create an inventory and define your hosts. This can be an INI file which might look like:

{{< accordion id="ini" >}}
    {{< accordion-item header="INI Inventory Example" >}}

```INI
mail.example.com

[webservers]
foo.example.com
bar.example.com

[dbservers]
one.example.com
two.example.com
three.example.com
    {{< /accordion-item>}}
{{< /accordion >}}

Or a YAML file, which I use, looks like:

{{< accordion id="yaml" >}}
    {{< accordion-item header="YAML Inventory Example" >}}
```YAML
all:
  hosts:
    mail.example.com:
  children:
    webservers:
      hosts:
        foo.example.com:
        bar.example.com:
    dbservers:
      hosts:
        one.example.com:
        two.example.com:
        three.example.com:
    {{< /accordion-item>}}
{{< /accordion >}}

Here is what my current YAML inventory looks like where I have defined variables for my SSH key and `sudo` password and have used IP addresses for my machines.

{{< accordion id="my-yaml" >}}
    {{< accordion-item header="My YAML Inventory" >}}
```YAML
all:
  vars:
    # These are global variables, they can be overwritten at the host level
    ansible_user: liam
    # We can use variables by putting them in double curly brackets
    # these variables are assigned in a vault file and read here
    #ansible_ssh_pass: '{{ host_default_pass }}'
    # The "become_pass" is used from sudo commands
    ansible_become_pass: '{{ host_default_pass }}'
    ansible_ssh_private_key_file: '{{ host_ssh_key }}'
    ansible_python_interpreter: auto_silent
    # This little trick is to use short hostnames like myhost
    # instead of myhost.mynetwork.local, the domain can be overwritten
    # at the host level as well
    #ansible_host: '{{ inventory_hostname }}.{{ host_domain }}'
    #host_domain: 'mynetwork.local'
  hosts:
    # These are hosts on the global level, we can also group hosts as
    # seen below under "children"
  children:
    # These are host groups, we can use groups later to target
    # hosts as a group instead of individually
    # hosts without ips are defined in .ssh/config
    containers:
      vars:
        ansible_user: root
      hosts:
        pihole:
          ansible_host: 192.168.1.124
        mysql-server:
          ansible_host: 192.168.1.134
        postgres-server:
          ansible_host: 192.168.1.232
        dev-ct:
          ansible_host:
    virtual_machines:
      hosts:
        media-server:
          ansible_host: 192.168.1.218
        docker-vm:
          ansible_host: 192.168.1.184
        dev-vm:
          ansible_host: 192.168.1.214
    {{< /accordion-item>}}
{{< /accordion >}}

As demonstrated in the examples above, you can reference individual machines or groups of machines in Ansible. This feature is particularly useful when writing playbooks that need to execute tasks against a specific set of machines. For instance, you may have a task that updates a group of webservers or machines located in a particular region. Moving forward, it is recommended to switch to using hostnames instead of IPs. Additionally, you can leverage Terraform to dynamically create and populate your inventory, which can then trigger a playbook run. This approach ensures that your inventory is always up-to-date and eliminates the need for manual updates.

### Basic Playbook Example

The basic example below is slightly adapted from one of playbooks that I apply to my virtual machines. In this example the `docker-vm` host is the target. The playbook:

* Does an `apt update` and `upgrade`
* Installs the `qemu-guest-agent` for Proxmox
* Installs some other packages that are useful such as `p7zip`, `unzip` and `rsync`
* Install `pip` and then `docker` and `docker-compose` Python packages using `pip`
* Finally, install `fail2ban`

{{< accordion id="basic-playbook" >}}
    {{< accordion-item header="Basic Playbook Example" >}}
```YAML
- hosts: docker-vm
  tasks:
    - name: update and upgrade apt packages
    become: yes
    apt:
        upgrade: yes
        update-cache: yes
    - name: install qemu guest agent
    become: yes
    apt:
        name: qemu-guest-agent
    - name: install some useful packages
    become: yes
    apt:
        pkg: 
        - p7zip-full
        - unzip
        - rsync
    - name: install pip
    become: yes
    apt:
        name: python3-pip
    - name: install ansible dependency pip docker
    pip:
        name: 
        - docker
        - docker-compose
    - name: install fail2ban
    become: yes
    apt:
        name: fail2ban
    {{< /accordion-item>}}
{{< /accordion >}}

### Playbook example with roles

Below is an example of a larger playbook that I have used to configure my media server. This uses some roles;

* `base`: Does an `apt` update and upgrade and are the tasks defined above for the "Basic Playbook Example"
* `user`: Sets up my user and configures `/etc/sudoers`
* `nfs_shares`: Add my NFS shares
* `ffmpeg`: Installs and configures ffmpeg
* `docker`: Install docker engine and docker compose
* `deploy_watchtower`: Deploys a [watchtower](https://containrrr.dev/watchtower/) container that monitors and notifies of Docker Image updates for my running containers
* `rclone`: Installs and configures rclone for backups

The deployment and configuration of my media server have become much easier with the use of Ansible. The tasks involved in this process include creating the necessary directories, copying over docker-compose files for media services, and any environment files. Once these tasks are completed, the containers are run, and their status is confirmed. This streamlined process saves a significant amount of time compared to manually copying over files and running commands on the terminal, which I have done for years. By combining Ansible with Terraform, I can now provision a virtual machine and configure my media server with ease by simply running the playbook against the machine. That way I can sit back and drink more :coffee:.

{{< accordion id="media-playbook" >}}
    {{< accordion-item header="Media Server Playbook Example" >}}
```YAML
- hosts: media-server
  roles:
    - base
    - user
    - nfs_shares
    - ffmpeg
    - docker
    - deploy_watchtower
    - rclone
  tasks:
    - name: creating required directories
      file:
        path: "/home/{{ ansible_user }}/docker/{{ item }}"
        state: directory
      loop:
          - jellyfin
          - media-stack
    - name: copy docker compose files
      copy:
        src: "templates/docker-compose/{{ item.file }}"
        dest: "/home/{{ ansible_user }}/docker/{{ item.name }}/docker-compose.yaml"
      loop:
        - { name: "jellyfin", file: "jellyfin.yaml" }
        - { name: "media-stack", file: "media-stack.yaml" }
    - name: copy env files
      copy:
        src: "templates/env/{{ item.file }}"
        dest: "/home/{{ ansible_user }}/docker/{{ item.name }}/.env"
      loop:
         - { name: "jellyfin", file: "jellyfin.env" }
         - { name: "media-stack", file: "media-stack.env" }
        
    - name: run media containers
      community.docker.docker_compose:
        project_src: "/home/{{ ansible_user }}/docker/{{ item }}/"
      loop:
        - jellyfin
        - media-stack
      register: output

    - ansible.builtin.debug:
        var: output

    - ansible.builtin.assert:
        that:
          - "{{ item }}.state.running"
        loop:
          - sonarr
          - radarr
          - nzbget
          - jellyseer
          - bazarr
          - prowlarr
          - jellyfin
          - readarr
    {{< /accordion-item>}}
{{< /accordion >}}

I have used variables in my inventory, playbooks and roles to either set default values or to access secrets. I won't cover variables here but will refer you to the [Variables Docs](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html) on using variables in your files, where to store variables for hosts or groups and how Ansible defines variable precendence. You can find more examples of the Ansible playbooks and roles I've created for my homelab (and also Terraform examples) in my [homelab GitHub repo](https://github.com/liamtill/homelab).

### Secrets

Similar to Terraform you can define secrets in a file, use an external secrets manager or even your CI/CD platform. To get stared I have opted to use [Ansible Vault](https://docs.ansible.com/ansible/latest/vault_guide/index.html) to store and encrypt any credentials and secret information. To do this I have stored and encrypted the relevant secrets for a machine in variables files. To create an encypted file you can run `ansible-vault create --vault-id test@multi_password_file file.yml` where `test@multi_password_file` is the password for the `test` vault from the `multi_password_file`. Your default text editor will open and you can input your secrets using YAML

```yaml
variable_name: secret_value
```

Alternatively, you can create a file with your secrets in and encrypt this existing file using: `ansible-vault encrypt file.yml`. You can view and edit encrypted files using `ansible-vault view file.yml` and `ansible-vault edit file.yml`.

## Conclusion

Ansible is a powerful and versatile IT automation tool that can help streamline your software engineering projects and improve overall efficiency. By leveraging its key features, such as playbooks, roles, and modules, you can simplify complex tasks, enhance collaboration, and ensure consistent, high-quality deployments. If you're looking to automate your infrastructure and application management, Ansible is an excellent choice to consider. You can already see from even the basic examples I've given here, and I'll cover how I have automated the deployment of my homelab in a future post, that utilising tools such as Ansible greatly increases efficiency and reduces errors.

## Resources and More Info

* [How Ansible Works](https://www.ansible.com/overview/how-ansible-works)
* [Ansible Documentation](https://docs.ansible.com/ansible/latest/index.html)
* [Ansible Installation](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
* [Playbooks](https://docs.ansible.com/ansible/latest/playbook_guide/index.html)
* [Modules](https://docs.ansible.com/ansible/latest/module_plugin_guide/index.html)
* [Templating](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_templating.html)
* [Roles](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html)
* [Inventory](https://docs.ansible.com/ansible/latest/inventory_guide/index.html)
* [Variables](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html)
* [Ansible Galaxy](https://galaxy.ansible.com/home)
* [Ansible Vault](https://docs.ansible.com/ansible/latest/vault_guide/index.html)
