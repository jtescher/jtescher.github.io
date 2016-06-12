---
layout: post
title:  "OpenShift"
date:   1970-01-01 00:00:00
categories:
---

## Installation

1. Install vagrant from [VagrantUp.com](https://www.vagrantup.com/)
1. Install the `vagrant-host-route` plugin via `$ vagrant plugin install vagrant-host-route`
1. In a new directory, create a file called `Vagrantfile` with the following contents:

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.require_version ">= 1.7.2"

Vagrant.configure("2") do |config|

  config.vm.box = "thesteve0/openshift-origin"           # Use the hashicorp openshift-origin base box
  config.vm.box_check_update = false                     # Ignore updates for now
  config.vm.network "private_network", ip: "10.2.2.2"    # Configure private network
  config.vm.hostname = "origin"                          # Set hostname to "origin"

  # Allow direct access to kubernetes services
  config.vm.provision :host_route do |host_route|
    host_route.destination = "172.30.0.0/16"
    host_route.gateway = "10.2.2.2"
  end

  # Set up vm
  config.vm.provider "virtualbox" do |vb|
     vb.memory = "5120"          # Use 5GB ram
     vb.cpus = 2                 # Use 2 cpus
     vb.name = "origin-1.2.0"    # Currently based on the 1.2.0 release
  end

end
```

Then to start the OpenShift VM, run `$ vagrant up` in this directory.

You can now view the sample application running at
[frontend-sample.apps.10.2.2.2.xip.io](http://frontend-sample.apps.10.2.2.2.xip.io/)
