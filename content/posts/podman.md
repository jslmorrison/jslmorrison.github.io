+++
title = "Podman"
date = 2023-05-21
tags = ["podman", "fedora-silverblue"]
categories = ["devops", "podman"]
draft = false
+++

[Podman](https://podman.io/) is installed out of the box on Fedora Silverblue. When I need to start new projects, podman coupled with ansible are the tools I use to configure them. Podman is a drop in replacement for Docker but I encountered a couple of issues. Read on to find out more.

<!--more-->

> Podman is an open source container, pod, and container image management engine. Podman makes it easy to find, run, build, and share containers.

Because Podman runs as the current user (not root) access to privileged ports &lt; 1024 will be denied. In order to enable access to port numbers lower than this we need to add some configuration as follows:

```cfg
# /etc/sysctl.d/podman-privileged-ports.conf
# default: 1024
net.ipv4.ip_unprivileged_port_start=80
```


## podman-compose {#podman-compose}

I’ve moved away from this to using ansible playbook to provision containers, but using this tool allows you to provision them using docker-compose files. It can be installed on the host with pip3:

```bash
pip3 install --user podman-compose
```
