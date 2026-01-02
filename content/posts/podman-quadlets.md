+++
title = "Podman Quadlets: Declarative Container Management with systemd"
date = 2026-01-01
tags = ["podman", "containers", "systemd"]
draft = false
+++

How to manage your containerized applications as effortlessly as any other system service on Fedora Silverblue. Podman quadlets bridges the gap between containers and systemd by allowing you to define container configurations declaratively in simple unit files - enabling automatic lifecycle management, seamless restarts and optimal resource control turning containers into first class citizens of your system's init process.

<!--more-->

> Quadlet is a tool for running Podman containers under systemd in an optimal way by allowing containers to run under systemd in a declarative way.

## Old way - playbook
Having previously created an Ansible playbook to start Traefik as the proxy for my local development environment, I wanted a solution to have it run automatically on every reboot as I got tired of having to do it manually or sometimes forgot.

```yaml
---
- name: Podman Traefik reverse proxy
  hosts: localhost

  tasks:
    - name: Create network
      containers.podman.podman_network:
        name: traefik-net
        state: present

    - name: Create traefik container
      containers.podman.podman_container:
        image: docker.io/library/traefik:3.6.5
        name: traefik
        network: traefik-net
        ports:
          - 80:80
          - 443:443
          - 8080:8080
        restart_policy: unless-stopped
        state: started
        volume:
          - "{{ playbook_dir }}/config/static.yml:/etc/traefik/traefik.yml:Z"
          - "{{ playbook_dir }}/config/dynamic:/etc/traefik/dynamic:Z"
```

With that in place, I would then have to `cd` to playbook directory and run it manually with:

```bash
ansible-playbook playbook.yaml
```

whenever I required it. Fine, but could be better.

## New way - Quadlet

Create a unit file that corresponds to the previous playbook content:

```ini
# ~/.config/containers/systemd/traefik.container
[Unit]
Description=Traefik Proxy
After=network-online.target
Wants=network-online.target

[Container]
Image=docker.io/library/traefik:3.6.5
ContainerName=traefik
Network=traefik-net
PublishPort=80:80
PublishPort=443:443
PublishPort=8080:8080
Mount=type=bind,src=/path/to/traefik-proxy/config/static.yml,destination=/etc/traefik/traefik.yml,Z
Mount=type=bind,src=//path/to/traefik-proxy/config/dynamic,destination=/etc/traefik/dynamic,Z

[Service]
Restart=always

[Install]
WantedBy=default.target
```

*Podman desktop also has handy functionality to generate these files from running container(s) for you.*

Check it works as expected by:
- stop services started via playbook ( if not already done so )
- execute following commands:
```bash
systemctl --user daemon-reload
systemctl --user restart traefik
```

and finally - if all good, reboot and Traefik service should be running as expected.

See here for more info:
- https://www.redhat.com/en/blog/quadlet-podman
- https://podman-desktop.io/
