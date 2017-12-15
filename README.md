# altdemo-vagrant
Vagrant topology for ALT Linux build for Mellanox Spectrum Switch and RHEL/CentOS servers.
Based on https://github.com/CumulusNetworks/cldemo-vagrant.
## Overview
VMs:
* **oob-mgmt-server** - management VM
* **oob-mgmt-switch** - MGMT switch to provide internal MGMT network (ALT)
* **leaf01-04** - leaf switches (ALT)
* **spine01-02** - spine switches (ALT)
* **server01-02** - servers (CentOS)

Host HW requirements:
* CPU: 2-4 cores/vCPU
* RAM: 8Gb minimal, 16Gb recommended
* Disk: 25Gb

## Network topology
ToDo

## Funtionality
ToDo

# Usage
* host# `vagrant up`
* host# `vagrant ssh oob-mgmt-server`
* oob-mgmt-server# `sudo su - admin`
* Now you have key-based ssh access to all VMs: `ssh leaf01`
