---
title: "Running Rootless Docker Alongside Podman on Fedora Silverblue"
date: 2026-06-01
draft: false
# description: "Run rootless Docker alongside Podman on Fedora Silverblue and sandbox coding agents like OpenCode using docker sbx."
# summary: "A practical guide to running rootless Docker in user space on Silverblue, alongside Podman, with a real-world sandbox use case using docker sbx."
tags: ["fedora-silverblue", "docker", "podman"]
# categories: ["containers", "development"]
# toc: true
---------

<!-- ## Introduction -->

If you’re already using Fedora Silverblue, you’re likely comfortable with its container first workflow and the role Podman plays as the default runtime. But even in a Podman native environment, there are still cases where **Docker compatibility matters** — especially when working with tools that rely on Docker specific behavior.

<!--more-->

Rather than modifying the base system, a cleaner approach is to run **Docker in rootless mode entirely in user space**, alongside Podman. This keeps the host immutable, avoids introducing a privileged daemon and aligns with the overall design philosophy of Silverblue.

In this guide, we’ll focus on exactly that: running **rootless Docker without touching the base image**, coexisting cleanly with Podman. More importantly, we’ll explore why this setup is useful in practice by walking through a real-world scenario - using Docker as a sandbox for a coding agent like OpenCode.

---

## Why Run Docker *and* Podman?

### Podman strengths

* Daemonless architecture (no root daemon)
* First class systemd integration
* Native rootless support
* Tight alignment with Fedora and Red Hat ecosystems

### Docker strengths

* Massive ecosystem support
* Tools that rely on the Docker socket/API
* Better compatibility with some developer tooling and CI systems

### The sweet spot

Running **Podman for day-to-day containers** and **rootless Docker for compatibility workloads** gives you:

* Security (no root daemon)
* Flexibility
* Compatibility
* Stability (immutable host stays untouched)

---

## Installing Rootless Docker (User-Space Only)

```bash
curl -fsSL https://get.docker.com/rootless | sh
```


### Configure environment

```bash
export PATH=$HOME/bin:$HOME/.local/bin:$PATH
```

Add to your shell profile for persistence.


### Start Docker

```bash
systemctl --user enable --now docker
```


### Verify

```bash
docker run --rm hello-world
```

---

## Using Podman Alongside Docker

No changes required — Podman remains your default runtime.

```bash
podman run quay.io/podman/hello
docker run hello-world
```

Both runtimes are fully isolated from each other.

## Why Rootless Matters

### Security

* No privileged daemon
* Reduced attack surface

### Silverblue alignment

* No base image mutation
* Fully user-space tooling

### Experimentation

* Safe for untrusted workloads
* Easy to reset

---

## Real Use Case: Running OpenCode in a Docker Sandbox

A coding agent like **OpenCode** can:

* Execute arbitrary code
* Install dependencies
* Modify files

This makes isolation essential.


## Why docker sbx?

`sbx` provides:

* Ephemeral environments
* Secure defaults
* Simple UX

## Installing `sbx`

Download release from https://github.com/docker/sbx-releases/releases to `/tmp`, extract contents, `cd` into extracted directory and run install script. Ensure install location is added to your `$PATH`. Start a new terminal session and run the following to verify it works.

```bash
sbx version
```

## Running OpenCode

```bash
sbx run opencode
```

## What You Get

* Rootless execution
* Ephemeral containers
* Filesystem isolation
* Process containment


## Why Not Just Use `docker|podman run`?

| Concern     | Manual       | `sbx`       |
| ----------- | ------------ | ----------- |
| Isolation   | Manual flags | Built-in    |
| Cleanup     | Manual       | Automatic   |
| DX          | Verbose      | Simple      |
| Consistency | Variable     | Predictable |


## Architecture Overview

```text
Silverblue (immutable host)
└── Rootless Docker (user)
    └── sbx sandbox
        └── OpenCode agent
```

---

## Final Thoughts

This hybrid model gives you:

* Podman → native workflows
* Docker → compatibility
* sbx → safe execution
* Silverblue → stability

Together, they form a **secure, flexible, and reproducible development environment**—ideal for modern AI-assisted workflows.

Happy hacking.
