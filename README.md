# RPM Tools

This is a build repo for the [Snoopy](
https://github.com/a2o/snoopy) project.

It provides a VagrantFile for setting up a
local rpm build environment.

# Vagrant Setup

This directory contains a VagrantFile which will allow you to easily
boot up a VirtualBox VM fully configured for local rpm building.

Make sure you have the latest [VirtualBox](virtualbox.org) installed
and [Vagrant](vagrantup.com) installed.

To download the base OS, setup the VM, install required packages, and
configure the system simply do: `vagrant up`

You can easily ssh into the vm by typing `vagrant ssh`.

When the build is complete rpms will be transfered to artifacts/
