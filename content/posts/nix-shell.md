+++
title = "Nix-shell"
date = 2023-10-11
tags = ["nixos", "nix"]
categories = ["nixos"]
draft = false
+++

According to [the manual](https://nixos.org/manual/nix/stable/command-ref/nix-shell):

> The command nix-shell will build the dependencies of the specified derivation, but not the derivation itself. It will then start an interactive shell in which all environment variables defined by the derivation path have been set to their corresponding values, and the script $stdenv/setup has been sourced. This is useful for reproducing the environment of a derivation for development.

How does that help me?

<!--more-->


## Ephemeral shells {#ephemeral-shells}

As an example - I need to search local directories quickly for specific string:

```bash
ag 'nix-shell' .
```

but I don't have the [silver-searcher](https://geoff.greer.fm/ag/) installed. Running the above will result in:

```bash
The program 'ag' is not in your PATH. You can make it available in an
ephemeral shell by typing:
  nix-shell -p silver-searcher
```

Using that suggestion, once it completes, I will be dropped into a temporary shell where i can repeat the search and it will work as expected.
Exit the shell and the silver-searcher will be gone.


## Development environments {#development-environments}

As an alternative to setting up and running a containerised development environment using docker or podman etc.


### E.g. setting up and running a [Symfony](https://symfony.com/) project {#e-dot-g-dot-setting-up-and-running-a-symfony-project}

Given i have already installed the [Symfony cli tool](https://github.com/symfony-cli/symfony-cli)

```bash
symfony check:requirements
```

will result in

```bash
no PHP binaries detected
```

create an ephemeral shell that has php binary:

```bash
nix-shell -p php
```

check it is available:

```bash
php -v

PHP 8.1.24 (cli) (built: Sep 26 2023 23:43:49) (NTS)
Copyright (c) The PHP Group
Zend Engine v4.1.24, Copyright (c) Zend Technologies
    with Zend OPcache v8.1.24, Copyright (c), by Zend Technologies
```

create new symfony project:

```bash
symfony new symfony-sandbox
```

```bash
[OK] Your project is now ready in /home/john/development/web/symfony-sandbox
```

In order to have the changes persist after exiting this nix-shell you need to write requirements to file

```bash
touch shell.nix
```

add the following content:

```nix
{ pkgs ? import <nixpkgs> {} }:
  pkgs.mkShell {
    # nativeBuildInputs is usually what you want -- tools you need to run
    nativeBuildInputs = with pkgs.buildPackages; [
      php81
    ];
}
```

and exit the temporary shell.

Should be back in a normal terminal shell (check prompt differences between the two). Try and run any of the symfony-cli commands then you'll get the 'no PHP binaries detected' error again. Not to worry:

```bash
nix-shell
```

then

```bash
symfony server:start
```

```nil
 [OK] Web server listening
      The Web server is using PHP FPM 8.1.24
      http://127.0.0.1:8000
```


### Bonus - using direnv {#bonus-using-direnv}

Add a `.envrc` file to project root directory with following content:

```bash
use nix
```

Install [direnv](https://direnv.net/) by adding it to package list in main configuration file; E.g.:

```nix
with pkgs; [
    direnv
]
```

add the following to `programs.zsh` section

```nix
initExtra = ''
    eval "$(direnv hook zsh)"
''
```

rebuild config

```bash
sudo nixos-rebuild switch
```

Then from the same directory that contains the `.envrc` file:

```bash
direnv allow .
```

Now anytime you `cd` in/out of project dir the environment shell will be loaded and unloaded automatically.

[See here](https://nixos.wiki/wiki/Development_environment_with_nix-shell) for more info.
