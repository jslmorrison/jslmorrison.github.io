+++
title = "Using Toolbx on Fedora Silverblue"
date = 2026-08-26
tags = ["fedora-silverblue"]
categories = ["linux", "fedora"]
draft = false
+++

Fedora Silverblue is an immutable operating system, so I try to avoid layering packages onto the host unless they are needed by the system itself. For command-line tools, [Toolbx](https://containertoolbx.org/) provides a convenient alternative: it gives me a familiar Fedora environment with `dnf`, while keeping those packages outside the host image.

This post shows how to create a Toolbx container, install Neovim in it, and launch Neovim from the host with a small Zsh alias.

<!--more-->

## Create a Toolbx container {#create-a-toolbx-container}

Toolbx is included with Fedora Silverblue. Create a container with:

```bash
toolbox create neovim
```

Enter it with:

```bash
toolbox enter neovim
```

Once inside, the prompt changes to show that you are working in the container. The container has access to your home directory, so your projects and configuration files remain available without copying them into the container.

## Install Neovim in Toolbx {#install-neovim-in-toolbx}

Install Neovim using the package manager from inside the Toolbx container:

```bash
sudo dnf install -y neovim
```

Check that the installation completed successfully:

```bash
nvim --version
```

This installs Neovim in the `neovim` container, not in the immutable Silverblue host. Other command-line tools can be installed in the same way, and can be removed or updated without changing the host image.

Leave the container with `exit` when you are finished:

```bash
exit
```

## Run Neovim from the host {#run-neovim-from-the-host}

Toolbx can run a command in a named container without opening an interactive shell. First, check that Neovim works from the host:

```bash
toolbox run -c neovim nvim --version
```

To avoid typing the full command each time, add this alias to `~/.zshrc` on the host:

```zsh
alias nvim='toolbox run -c neovim nvim'
```

Reload the configuration:

```bash
source ~/.zshrc
```

You can now open a file in the current directory with:

```bash
nvim path/to/file
```

Arguments are passed through to Neovim, so commands such as these work as well:

```bash
nvim .
nvim -u NONE path/to/file
```

## Updating the container {#updating-the-container}

Update the packages in the container whenever needed:

```bash
toolbox run -c neovim sudo dnf upgrade
```

The container is separate from the host deployment, but it is still a normal Fedora environment and should be kept updated. If you no longer need it, remove it with:

```bash
toolbox rm neovim
```

For a list of available containers, use `toolbox list`.