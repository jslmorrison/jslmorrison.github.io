+++
title = "Ansible"
date = 2023-05-20
tags = ["ansible", "fedora-silverblue"]
categories = ["devops", "ansible"]
draft = false
+++

Since I have been using Fedora Silverblue full time I have switched away from using [Docker](https://www.docker.com/) and [docker-compose](https://docs.docker.com/compose/), for container orhestration, to creating [ansible playbooks](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_intro.html) to provision local development containers.

<!--more-->

> Ansible is an open source, command-line IT automation software application written in Python. It can configure systems, deploy software, and orchestrate advanced workflows to support application deployment, system updates, and more.

Ansible playbooks are expressed in yaml format and can be used to, but not limited to, orchestrate steps of any manual ordered process, on multiple sets of machines (local or remote), in a defined order. In my case, I have generally been using them to replace tasks defined in `docker-compose` files.

Python is included in the base Silverblue install, but we will need to install `pip3` in order to be able to install required python tools/packages in user filesystem, which is writeable. To install execute the following:

```bash
python -m ensurepip --user --upgrade --force-reinstall
```

The pip3 binary will now be available on the host. The binary is installed to `~/.local/bin`. Ensure that directory is added to your $PATH.

Ansible can now be installed, just run the following in the terminal:

```bash
pip3 install --user ansible
```

Here is an example playbook, from the [podman container module docs](https://galaxy.ansible.com/containers/podman) of the [Ansible galaxy](https://galaxy.ansible.com/home) community that will result in a [Redis](https://redis.com/) server intsalled inside a [Podman](https://podman.io/) container:

```yaml
# example-redis-playbook.yaml
---
- name: Using Podman collection
  hosts: localhost
  tasks:
    - name: Run redis container
      containers.podman.podman_container:
        name: myredis
        image: redis
        command: redis-server --appendonly yes
        state: started
        recreate: yes
        expose:
          - 6379
```

Run the playbook from within th etermainal as follows:

```bash
ansible-playbook example-redis-playbook.yaml
```


## Ansible galaxy {#ansible-galaxy}

> Jump-start your automation project with great content from the Ansible community. Galaxy provides pre-packaged units of work known to Ansible as roles and collections.

A great community to find content for provisioning infrastructure and deploying applications for use in your playbooks.
