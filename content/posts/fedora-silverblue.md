+++
title = "Fedora Silverblue"
date = 2023-05-19
tags = ["fedora-silverblue"]
categories = ["linux", "fedora"]
draft = false
+++

I've been using linux as my OS of choice for many years now, for personal and work related usage and my current distro of choice is [Fedora Silverblue](https://fedoraproject.org/silverblue/). If you are wondering what the difference is compared to the regular Fedora workstation then read on.

<!--more-->

> Fedora Silverblue is an immutable desktop operating system. It aims to be extremely stable and reliable. It also aims to be an excellent platform for developers and for those using container-focused work-flows.

The user interface and the experience remains unchanged from a typical Fedora Workstation release but things are different when it comes to installing and managing applications and setting up a local development environment. Applications can be installed as `flatpaks` or layered onto the base OS image. I want to keep layering to very minimum and currently only have 2 layered packages:

-   zsh
-   gstreamer plugins

Everything else I need is installed as a flatpak, via [flathub](https://flathub.org/), or within a toolbx container, which I will go into in more detail in following posts.

I dont need to worry about keeping my system updated either, just boot into latest image and thats all. If in the unlikely event that the currently booted image has a problem, then I can rollback to previous image. Keeping layered packages to a minimum reduces the chances of problems arising when they get applied to latest image.

The following utlities are what I require to enable me to have a working dev environment:

-   [ansible]({{< relref "ansible" >}})
-   toolbx
-   pip3
-   [podman]({{< relref "podman" >}})
-   [traefik]({{< relref "traefik" >}})

I'll post more about installing those in other articles.


## Toolbx {#toolbx}

Toolbx comes pre-installed as part of the base OS layer and it provides a familiar package-based environment in which tools and libraries can be installed and used.

<!--more-->

> Toolbx makes it easy to use a containerized environment for everyday software development and debugging. On immutable operating systems, like Fedora Silverblue, it provides a familiar package-based environment in which tools and libraries can be installed and used.

[See docs for more info](https://containertoolbx.org/).


## Distrobox {#distrobox}

> Use any Linux distribution inside your terminal. Enable both backward and forward compatibility with software and freedom to use whatever distribution you’re more comfortable with. Distrobox uses podman or docker to create containers using the Linux distribution of your choice.

Similar to `toolbx`, simply put it’s a fancy wrapper around podman or docker to create and start containers highly integrated with the hosts.

Installed on host by cloning the repo and running the ’install’ shell script, which installs the binary in `~/.local/bin`. Nothing else required.

[See docs for more info.](https://distrobox.privatedns.org/)
