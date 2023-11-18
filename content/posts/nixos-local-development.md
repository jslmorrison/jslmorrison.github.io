+++
title = "Local development environments on NixOS"
date = 2023-11-18
tags = ["nixos"]
categories = ["nixos"]
draft = false
+++

After spending some time researching possible solutions for non containerised development environments on [NixOS](https://nixos.org/), I have finally settled on [Devbox](https://www.jetpack.io/devbox) from [Jetpack](https://www.jetpack.io/).
This enables me to configure required services per project, quickly and easily, on any machine.

<!--more-->

> Devbox creates isolated, reproducible development environments that run anywhere. No Docker containers or Nix lang required


## Installing Devbox {#installing-devbox}

Use the following install script to get the latest version of Devbox:

```bash
curl -fsSL https://get.jetpack.io/devbox | bash
```

Read the [quickstart docs](https://www.jetpack.io/devbox/docs/quickstart/) for an overview of what's available. Read on here for configuring a sample [Symfony](https://symfony.com/) web project.


## Create a new Symfony project {#create-a-new-symfony-project}

-   Start a new nix shell with the minimum package requirements

<!--listend-->

```bash
nix-shell -p php82 php82Packages.composer symfony-cli
```

Then from inside the `nix-shell` create a new Symfony project using the `symfony-cli` tool:

```bash
symfony new symfony-project
```

-   Exit the nix-shell and switch into the newly created project directory
-   Init the devbox

<!--listend-->

```bash
devbox init
```

-   Add the required devbox services

<!--listend-->

```bash
devbox add caddy curl php82 php82Packages.composer
```

-   At this point we can open a `devbox shell` and confirm we have the expected php and composer versions

<!--listend-->

```bash
php -v

PHP 8.2.7 (cli) (built: Jun  6 2023 21:28:56) (NTS)
Copyright (c) The PHP Group
Zend Engine v4.2.7, Copyright (c) Zend Technologies
    with Zend OPcache v8.2.7, Copyright (c), by Zend Technologies

composer --version

Composer version 2.6.5 2023-10-06 10:11:52
```

-   Before starting the devbox services, a couple of things need to be changed.
    Change the port caddy is listening on from `8082` to `8083` to avoid clashing with the php82fpm default port number, change the root to the `public/` directory of the project and add directive to serve php files.

    The caddy config file should now have the following content:

<!--listend-->

```cfg
# devbox.d/caddy/Caddyfile

# See https://caddyserver.com/docs/caddyfile for more details
{
	admin 0.0.0.0:2020
	auto_https disable_certs
	http_port 8800
	https_port 4443
}

:8083 {
	root * {$DEVBOX_PROJECT_ROOT}/public
    php_fastcgi 127.0.0.1:{$PHPFPM_PORT}

	log {
		output file {$CADDY_LOG_DIR}/caddy.log
	}
	file_server
}
```

You can now start the devbox services:

```bash
devbox services up
```

The project is now ready and available to browse on [http://localhost:8083/](http://localhost:8083/)

Further services, e.g. a database, can be added as required. See the excellent [documentation](https://www.jetpack.io/devbox/docs/) for more info.
