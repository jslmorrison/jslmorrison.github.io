+++
title = "Traefik"
date = 2023-05-22
tags = ["traefik", "fedora-silverblue"]
categories = ["devops", "traefik"]
draft = false
+++

Setting up a local dev environment with [Traefik](https://doc.traefik.io/traefik/) as the proxy has some benefits:

-   highly configurable
-   only need to expose required ports on the Traefik container
-   simplifies configuring SSL certificates and redirects for each dev site.

Given the benefits, I found it worth while investing the time and effort in learning enough about Traefik to apply those benefits to new projects as it simplifies config for each new project.

<!--more-->

I had done the same when I was running Docker on a regular Fedora workstation system and needed to have it replicated with podman containers in Fedora Silverblue system.
Setup seemed simpler when using docker containers, as Traefik has built in auto-discovery available via container labels. Podman containers need to be handled differently though.


## Local dev env with Podman and Traefik {#local-dev-env-with-podman-and-traefik}

As usual, an `ansible-playbook` is used to configure required containers:

```yaml
# Example ansible-playbook with file provider
---
- name: Podman Traefik dev box
  hosts: localhost

  tasks:
    - name: Create dev box network
      containers.podman.podman_network:
        name: podman-traefik-network
        state: present

    - name: Create Traefik container
      containers.podman.podman_container:
        name: podman-traefik
        image: traefik:v2.10.1
        ports:
          - 80:80
          - 443:443
          - 3306:3306
          - 8080:8080
        network: podman-traefik-network
        state: started
        volume:
          - "{{ playbook_dir }}/tools/traefik/static-config.yml:/etc/traefik/traefik.yml:Z"
          - "{{ playbook_dir }}/tools/traefik/dynamic/project1.yml:/etc/traefik/dynamic/project1.yml:Z"
```

```yaml
# static-config.yml
---
global:
  checkNewVersion: true
  sendAnonymousUsage: true

entryPoints:
  web:
    address: :80
  websecure:
    address: :443
  mariadb:
    address: :3306

log:
  level: DEBUG
  filePath: log/traefik.log
  format: json

accessLog:
  filePath: log/traefik-access.log
  format: json

api:
  insecure: true
  dashboard: true

providers:
  file:
    directory: /etc/traefik/dynamic
    watch: true
```

The Traefik dashboard will be available on <http://localhost:8080>

For each local dev site that you want to incorporate with Traefik, add a dynamic config file as follows:

```yaml
---
http:
  routers:
    project1-https-router:
      rule: "Host(`project1.localhost`)"
      service: project1
      tls: {}

    project1-http-router:
      rule: "Host(`project1.localhost`)"
      middlewares:
        - https-redirect
      service: project1

  services:
    project1:
      loadBalancer:
        servers:
          - url: http://project1-web:80
            # project1-web in url here should match service defined in the project1 container

  middlewares:
    https-redirect:
      redirectScheme:
        scheme: https
        permanent: true
        port: "443"
```

<http:///project1-web> will be redirected to use https and the ssl certificate provided by Traefik or preferrably create your own using a tool like `mkcert` for example.

I'll write up more about using mkcert for this purpose in a later post.
