+++
title = "NixOS"
date = 2023-10-10
tags = ["nixos"]
categories = ["nixos"]
draft = false
+++

I've been reading a lot about [NixOS](https://nixos.org/) lately and wanted to learn more so I've took the plunge and now using it as my daily driver. Hopefully I'll learn more and get a better understanding of it by actually using it.
Not sure if I'll keep it permanently or go back to [Fedora Silverblue](https://fedoraproject.org/silverblue/), but nothing ventured nothing gained.

> NixOS, a Linux distribution based on the purely functional package management system Nix, that is composed using modules and packages defined in the Nixpkgs project.

<!--more-->


## Installation {#installation}

Graphical installer was very straight forward, so nothing untoward to mention here.


## Configuration {#configuration}

> Nix is a tool that takes a unique approach to package management and system configuration. Learn how to make reproducible, declarative and reliable systems.

Configuration is declared in a single file `/etc/nixos/configuration.nix`
I've moved mine to my home directory and symlinked it to the default path though, as I wanted to manage my environment with [Home Manager](https://nixos.wiki/wiki/Home_Manager).


### Package installation {#package-installation}

Packages can now be installed by declaring them as follows:

```nix
home.packages = with pkgs; [
  firefox
  lazygit
  librewolf
]
```

and so on.

After making changes to the config run the following in terminal:

```bash
nixos-rebuild switch
```

to rebuild the system configuration and create a new [generation](https://nixos.wiki/wiki/Overview_of_the_NixOS_Linux_distribution#Generations).

The newly added packages are now available and ready to use.


## Goal {#goal}

My main goal will be to get the system configured and working to my liking, ultimately to have all development requirements defined per project via [nix-shell](https://nixos.org/manual/nix/stable/command-ref/nix-shell)'s or [nix develop](https://nixos.org/manual/nix/stable/command-ref/new-cli/nix3-develop) if I switch to using [flakes](https://nixos.wiki/wiki/Flakes).
