+++
title = "Schedule tasks in systemd"
date = 2023-06-05
tags = ["fedora-silverblue", "systemd"]
categories = ["fedora-silverblue"]
draft = false
+++

Given that `cron` is not installed by default on a Fedora Silverblue host, it can either be added as a layered package or you can use `systemd` which is available by default.

<!--more-->

As I want to keep layered packages to bare minimum, I have chose to utilise `systemd` to configure any scheduled tasks I may need.

Systemd requires minimum of 2 files to schedule task:

-   a service unit
-   and a timer unit
-   and optionally a script if command to run is not so simple

Files should be placed in `~/.config/systemd/user` directory, which may not exist yet. If need be, create it with:

```bash
mkdir -p ~/.config/systemd/user
```

As an example - I wanted to have the Traefik container that I [setup previously]({{< relref "traefik" >}}) as the proxy for my local development environment, to be started automatically.

```nil
# schedule-traefik.service
[Unit]
Description=Run script to start Traefik container

[Service]
Type=simple
ExecStart=%h/bin/schedule-traefik.sh
RemainAfterExit=true

[Install]
WantedBy=default.target
```

```nil
# schedule-traefik.timer
[Unit]
Description=Schedule timer to start Traefik container

[Timer]
#Run 60 seconds after boot for the first time
OnBootSec=60

[Install]
WantedBy=timers.target
```

```bash
# bin/schedule-traefik.sh
#!/bin/sh
cd $HOME/development/podman-traefik-dev-box/ && $HOME/.local/bin/ansible-playbook play.yaml
```

Once the _.service_ and _.timer_ unit files are in place, the service can be enabled with:

```bash
systemctl --user enable schedule-traefik.service
```

and manually tested with:

```bash
systemctl --user start schedule-traefik.service
```

Also check the output and that the container is running:

```bash
journalctl --user -fxeu schedule-traefik.service
```

```bash
podman ps -a
```

Reboot and check again. Container should be up and running as expected.
