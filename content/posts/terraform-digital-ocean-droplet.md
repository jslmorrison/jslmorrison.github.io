+++
title = "Automate cloud infrastructure with Terraform"
date = 2023-12-28
tags = ["terraform", "devops"]
draft = false
+++

How to use [Terraform](https://www.terraform.io/), an infrastructure as code (IaC) tool, to provision a [DigitalOcean](https://www.digitalocean.com/) droplet.

<!--more-->

> Terraform is a platform agnostic infrastructure as code (IaC) tool offered by HashiCorp. It allows users to build, change and version cloud and on-premises resources safely and efficiently.

The tool works with various cloud providers and other services through their APIs, and it uses a state file to keep track of all changes to the infrastructure.


## Define infrastructure {#define-infrastructure}

You define your infrastructure in a file ending with `.tf` extension, e.g `main.tf`. The content of this file is written in HashiCorp Configuration Language (HCL) and this is where you declaratively describe the desired infrastructure.

```hcl
terraform {
  # use the digitalocean provider
  required_providers {
    digitalocean = {
      source = "digitalocean/digitalocean"
      version = "~> 2.0"
    }
  }
}

# define required variables - a personal access token and ssh key to use
variable "do_token" {}
variable "pvt_key" {}

provider "digitalocean" {
  token = var.do_token
}

# a previously created ssh key added to your DO account
data "digitalocean_ssh_key" "terraform" {
  name = "my-do-key"
}

# create droplet
resource "digitalocean_droplet" "my-do-web" {
    image = "fedora-39-x64"
    name = "my-do-web"
    region = "lon1"
    size = "s-1vcpu-1gb"
    ssh_keys = [
      data.digitalocean_ssh_key.terraform.id
    ]

    connection {
      host = self.ipv4_address
      user = "root"
      type = "ssh"
      private_key = file(var.pvt_key)
      timeout = "2m"
    }
}

# create project and add droplet resource
resource "digitalocean_project" "my-do-web" {
  name = "My D.O. web"
  description = "Web app and api"
  purpose = "Web Application"
  resources   = [digitalocean_droplet.my-do-web.urn]
}

# Update firewall rules
resource "digitalocean_firewall" "my-do-web" {
  name = "web-ssh"

  droplet_ids = [digitalocean_droplet.my-do-web.id]

  inbound_rule {
    protocol = "tcp"
    port_range = "22"
    source_addresses = ["0.0.0.0/0", "::/0"]
  }

  inbound_rule {
    protocol = "tcp"
    port_range = "80"
    source_addresses = ["0.0.0.0/0", "::/0"]
  }

  inbound_rule {
    protocol = "tcp"
    port_range = "443"
    source_addresses = ["0.0.0.0/0", "::/0"]
  }

  outbound_rule {
    protocol = "tcp"
    port_range = "53"
    destination_addresses = ["0.0.0.0/0", "::/0"]
  }

  outbound_rule {
    protocol = "udp"
    port_range = "53"
    destination_addresses = ["0.0.0.0/0", "::/0"]
  }
}
```

See the [DigitalOcean provider documentation](https://registry.terraform.io/providers/digitalocean/digitalocean/latest/docs) for more info.


## Apply the infrastructure {#apply-the-infrastructure}

Run the following command in the terminal from the same directory as the config file:

```bash
terraform apply -var "do_token=${DO_PAT}" -var "pvt_key=$HOME/.ssh/id_rsa"
```

If you browse to your digitalocean dashboard now, you will see the resources (project and droplet) as defined above.
