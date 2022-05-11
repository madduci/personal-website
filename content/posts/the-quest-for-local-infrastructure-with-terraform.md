+++
title = "The quest for local Infrastructure as Code with Terraform"
date = 2022-12-31
ref = "local-ifrastructure-as-code-terraform"
tags = ["terraform", "tools", "devops", "infrastructure"]
categories = ["tools", "terraform"]
draft = true
+++

Every person that works in a DevOps role or deals with Cloud Infrastructure (you know, AWS, GCP, Azure and so on) has read at least once about the concept of Infrastructure as Code and as probably used a magical tool that can help them achieving it: [**terraform**](https://www.terraform.io/).

Quoting the original website,

> Terraform is a tool for building, changing, and versioning infrastructure safely and efficiently. Terraform can manage existing and popular service providers as well as custom in-house solutions.

Terraform is really extensible thanks to the concept of _terraform providers_, plugins that allow engineers to deal with different resources, including data sources, cloud providers and so on. It bases also its fortune on avoiding the [cloud vendors lock-in](https://www.cloudflare.com/learning/cloud/what-is-vendor-lock-in/), allowing to deploy the infrastructure on multiple providers, without getting surprises.

But...how easy is to manage custom in-house solutions, e.g. our local private server? In this case, the choices can be probably counted with a single hand, since the terraform providers available  for local infrastructure are mostly community projects, not directly available in the official Terraform Registry, and sometimes lack of support or are out of date, due to recent changes in terraform itself.
To make things even more complicated, the community providers have to be installed manually, following a specific path:

**TODO**: add snippet

## The Plan

Thinking about local infrastructure, let's assume we have a system with enough disk space and RAM available (it can be our Laptop or a Desktop machine). 
Let's assume we'd like to build up 2 scenarios:

1. deploy some Docker Images on our server and execute them (as alternative to `docker-compose`).
2. a small, local Kubernetes cluster (2 or 3 Nodes, depending on the hardware specs available) and deploy a demo application, packaged as a Docker Image, so we can run some local tests, before even deploying it on a cloud system.

What kind of tools can we use to realize Infrastructure as Code with terraform? 
Let's write some ideas and find out if the following terraform providers can help us reaching our goal:

* [VirtualBox](https://github.com/terra-farm/terraform-provider-virtualbox)
* [Vagrant](https://github.com/bmatcuk/terraform-provider-vagrant)
* [Docker](https://www.terraform.io/docs/providers/docker/index.html) 
* [Docker Machine](https://github.com/gstruct/terraform-provider-dockermachine)
* [Proxmox](https://github.com/Telmate/terraform-provider-proxmox) or [VMware ESXi](https://github.com/josenk/terraform-provider-esxi)

## Building a Machine with Docker using Terraform

I'll assume there's no Docker installed on our machine, therefore we want to build a virtual machine with Docker configured and ready to host our images. To make things easier, I'll assume that the only ports exposed are:

* 2222 for SSH access
* 2376 for Docker (required for remote connections)
* 7080 and 7443 for our Docker Application

### Using Community-driven Providers

The next subsections will explore the providers used to achieve our goal. Some of the providers are [community-driven](https://www.terraform.io/docs/configuration/providers.html#third-party-plugins), which means that they aren't available through the [Terraform Registry](https://registry.terraform.io), so eventually Terraform will show up initialization errors while trying to fetch them from the registry.

NOTE: Terraform 0.13+ is needed

In order to overcome this problem, we need to define first a `versions.tf` file, where we specify the provider(s):

```
terraform {
  required_providers {
    A_PROVIDER = {
       source = "terraform-providers/A_PROVIDER"
    }
    B_PROVIDER = {
       source = "terraform-providers/B_PROVIDER"
    }
    ...
    Z_PROVIDER = {
       source = "terraform-providers/Z_PROVIDER"
    }
  }
  required_version = ">= 0.13"
}
```

And then, create for each community provider a folder in the current Terraform project and install there the providers:

```
.terraform/plugins/registry.terraform.io/terraform-providers/A_PROVIDER/X.Y.Z/platform_name/provider-name
.terraform/plugins/registry.terraform.io/terraform-providers/B_PROVIDER/X.Y.Z/platform_name/provider-name
...
.terraform/plugins/registry.terraform.io/terraform-providers/Z_PROVIDER/X.Y.Z/platform_name/provider-name
```

where `X.Y.Z` is the version of the provider, `platform_name` is the platform you're using (e.g., linux_amd64) and the `provider-name` follows the convention **terraform-provider-NAME_vX.Y.Z**

### VirtualBox Terraform Provider

I'd like to build the machine with VirtualBox, which is widely available on major operating systems and it can be used for free, without additional licenses or costs. As for today, there's no official VirtualBox Terraform Provider on HashiCorp's website, so we must rely on community-based ones: one of the most active and promising providers for VirtualBox is the [Terra-Farm one](https://github.com/terra-farm/terraform-provider-virtualbox).

Quoting the README _Limitations_ section:

> Experimental provider!
> The defaults here are only tested with the vagrant insecure (packer) keys as the login.

My `versions.tf` is configured as follows:

```
terraform {
  required_providers {
    virtualbox = {
       source = "terraform-providers/virtualbox"
    }
  }
  required_version = ">= 0.13"
}
```

And my plugins folder contains the following information:

```
.terraform/plugins/registry.terraform.io/terraform-providers/virtualbox/0.2.0/linux_amd64/terraform-provider-virtualbox_v0.2.0
```

Following the usage example as in the [official website](https://terra-farm.github.io/provider-virtualbox/reference/resource_vm.html), the machine could be successfully created (after roughly 1 hour!) but couldn't be started, causing an internal error in Terraform, leaving also VirtualBox in a dirty state, so I consider the trial unsuccessful.

### Vagrant Terraform Provider

The second trial works directly with Vagrant, which was also internally used by the VirtualBox Provider, so why not going straight with it? The [Vagrant Provider](https://github.com/bmatcuk/terraform-provider-vagrant) is, as the previous one, a community-driven one, so, again, we have to install it locally, before using it.

My `versions.tf` is configured as follows:

```
terraform {
  required_providers {
    vagrant = {
       source = "terraform-providers/vagrant"
    }
  }
  required_version = ">= 0.13"
}
```

And my plugins folder contains the following information:

```
.terraform/plugins/registry.terraform.io/terraform-providers/vagrant/2.0.0/linux_amd64/terraform-provider-vagrant_v2.0.0
```

### Docker Provider

## Building a Kuberneters Cluster using Terraform

### Vagrant Terraform Provider

### Docker Machine Provider

### Proxmox Provider
