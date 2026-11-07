+++
title = "Nix package manager Toolbx"
date = 2026-11-07
tags = ["fedora-silverblue", "nix"]
categories = ["linux", "fedora"]
draft = false
+++

As an alternative to maintaining multiple Toolbx containers for each command-line tool, you can use one Toolbx and install and manage your packages with Nix. This approach keeps your tools off the Fedora Silverblue host while letting Nix handle package installation and updates in a single Toolbx container.

<!--more--> 

## One container with Nix package manager {#one-nix-container}

Having recently upgraded to Fedora Silverblue 45, I was looking for an easy way to install and use Nix as package manager rather than managing multiple toolboxes as I have been doing.

What I wanted was a simple method to install and use Nix without `sudo` or other workarounds and use that to manage required packages as and when needed.

Here is outline of necessary steps in order to enable that:

- Create a dedicated toolbox for Nix:

```bash
toolbox create nix
toolbox enter nix
sudo dnf install -y nix --exclude nix-daemon
```

That's it, couldn't be easier.

Examples of using it:
```bash
toolbox run -c nix nix run nixpkgs#lazygit
toolbox run -c nix nix run nixpkgs#neovim
```

or add an alias in your `.zshrc`

`alias nvim="toolbox run -c nix nix run nixpkgs#neovim"`

This setup combines the best of both worlds: Silverblue's clean, immutable host combined with Nix package management makes command-line tools easy to install and update in one place.
