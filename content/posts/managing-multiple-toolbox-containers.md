+++
title = "Managing Multiple Toolbx Containers"
date = 2026-08-27
tags = ["fedora-silverblue"]
categories = ["linux", "fedora"]
draft = false
+++

As mentioned in previous posts, Toolbx gives Fedora Silverblue a mutable environment for installing command-line tools without layering them onto the host. A single container is enough to get started, but my preference is to create a separate Toolbx container for each tool I use regularly.

<!--more-->

## One container per tool {#one-container-per-tool}

Create the containers with names that describe their purpose, e.g.:

```bash
toolbox create neovim
toolbox create hugo
toolbox create lazygit
```

Enter each container and install its package:

```bash
toolbox enter neovim
sudo dnf install -y neovim
exit
```

```bash
toolbox enter hugo
sudo dnf install -y hugo
exit
```

```bash
toolbox enter lazygit
sudo dnf install -y lazygit
exit
```

The packages are installed in their respective containers, rather than on the Silverblue host or in one large shared environment.

## Why keep tools separate? {#why-keep-tools-separate}

Putting every command-line tool in one container is convenient initially, but it couples unrelated tools together. A separate container for each tool has a few practical advantages:

-   **Smaller environments:** each container contains only the packages needed by its tool.
-   **Fewer dependency conflicts:** tools can use different package versions or supporting libraries without competing in one environment.
-   **Independent maintenance:** updating or recreating one container does not affect the others.
-   **Easier troubleshooting:** when a tool stops working, its container is a smaller and more obvious place to investigate.
-   **Simple removal:** deleting an unwanted tool means removing one container instead of cleaning it out of a shared environment.

This is a preference rather than a rule. Tools that are routinely used together may belong in the same container, especially when they need to share SDKs or development dependencies. The point is to avoid making one container an accidental collection of everything installed on the host.

## Update every container from one script {#update-every-container-from-one-script}

Separate containers are easier to maintain when their names are kept in one list. The following small Bash script updates them all:

```bash
#!/usr/bin/env bash

set -euo pipefail

# Fetch the list of Toolbx container names (skip header row)
containers=$(toolbox list --containers 2>/dev/null | awk 'NR>1 {print $2}')

if [[ -z "${containers}" ]]; then
  echo "No Toolbx containers found."
  exit 0
fi

for container in ${containers}; do
  echo "Updating Toolbx container: ${container}"
  toolbox run --container "${container}" sudo dnf -y update
  echo "Finished updating ${container}"
  echo
done

echo "All Toolbx containers have been processed."
```

Save it as `~/bin/update-toolboxes`, for example, make it executable, and run it whenever the containers need updating:

```bash
chmod +x ~/bin/update-toolboxes
~/bin/update-toolboxes
```

`set -euo pipefail` makes the script stop if a command fails, if an unset variable is used, or if a command in the pipeline fails. This prevents a partially successful update from looking like a completely successful one.


When a container is no longer needed, remove it independently:

```bash
toolbox rm whatever
```

Managing the containers as a small named set gives me the separation I want without turning updates into a manual chore.