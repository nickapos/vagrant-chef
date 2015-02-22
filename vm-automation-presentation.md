% Automated vm generation
% Nick Apostolakis
% 27th of February 2015

# A walkthrough of the technologies used in NCR Edinburgh

The goal is to create a fully configured vm artifact ready to be imported in the virtualization software of choice. We will use VirtualBox for the development system

  * [Packer](https://www.packer.io/) This is used for the creation of custom Vagrant boxes
  * [Berkshelf](https://downloads.chef.io/chef-dk/) Part of Chef Development toolkit. Used for defining cookbook dependencies
  * [Chef](https://downloads.chef.io/chef-dk/)
  * Vagrant-berkshelf The vagrant plugin that is used to transfer the cookbooks on the new vm
  * [Vagrant](https://www.vagrantup.com/) Vagrant will be used to generate the finished product

# Packer

Packer is an open source tool for creating identical machine images for multiple platforms from a single source configuration. Packer is lightweight, runs on every major operating system, and is highly performant, creating machine images for multiple platforms in parallel. Packer does not replace configuration management like Chef or Puppet. In fact, when building images, Packer is able to use tools like Chef or Puppet to install software onto the image.

A machine image is a single static unit that contains a pre-configured operating system and installed software which is used to quickly create new running machines. Machine image formats change for each platform. Some examples include AMIs for EC2, VMDK/VMX files for VMware, OVF exports for VirtualBox, etc.

Although nowadays Packer supports Chef itself, we are basically using its Vagrant post-processor to create custom Vagrant boxes from the net installation iso of Centos 6.6. Originally Packer did not support these provisioners, so we used Vagrant instead. Nowadays its feature list is quite close to the Vagrant

# Packer template file example

  * Templates are JSON files that configure the various components of Packer in order to create one or more machine images. 
  * Templates are given to commands such as packer build, which will take the template and actually run the builds within it, producing any resulting machine images.

Packer templates are often brittle and not so easy to read as Vagrant files. The following example retrieves the ubuntu server iso and prepares a system using shell as a provisioner and executing the script setup_things.sh

    {
    "builders": [
    {
      "type": "virtualbox-iso",
      "guest_os_type": "Ubuntu_64",
      "iso_url": "http://releases.ubuntu.com/12.04/ubuntu-12.04.5-server-amd64.iso",
      "iso_checksum": "769474248a3897f4865817446f9a4a53",
      "iso_checksum_type": "md5",
      "ssh_username": "packer",
      "ssh_password": "packer",
      "ssh_wait_timeout": "30s",
      "shutdown_command": "echo 'packer' | sudo -S shutdown -P now"
    }
    ],

    "provisioners": [
    {
      "type": "shell",
      "script": "setup_things.sh"
    }
    ]
    }

# Berkshelf
  * The Berkshelf is a tool that can be used to manage and define cookbooks dependencies.
  * Berkshelf is used by our vm creation tool Vagrant to download and install all the cookbooks that we need on the new vm
  * Nowadays it is part of Chef Development Kit (ChefDK)
  * The main configuration file of Berkshelf is a file called Berksfile

## Berkshelf commands

    berks 

will download the cookbooks and 

    berks vendor

will upload them to the new vm. You dont need to do this manually, there is a Vagrant plugin that takes care of everything

--------------------------------------

## Berksfile

There is a community cookbook repository and Berkshelf can acces them by using the following syntax:

    source "https://supermarket.chef.io"
    cookbook 'cookbook_name', "~> 2.6"

Chef supermarket is the directory of the community cookbooks.

Berkhelf can also use cookbooks from alternative locations (e.g git repos, github, gitlab etc) with a small change in the Berksfile syntax:

    cookbook 'packages', git: 'git@github.com:mattray/packages-cookbook.git'

# Chef

For the purposes of this presentation we are going to use a very simple cookbook, that takes a list of packages and installs it on the new vm. It is developed by a Chef employee Matt Ray and is distributed unde the Apache License. It does not matter which distribution we choose to use, it will work in all of them, Chef takes care of the differences in package managers.

Core chef concepts:
  
  * Cookbook attributes
  * The runlist

## Cookbook attributes

One important feature of attributes is that they can be over-riden during runtime, both from the calling scripts and the cookbooks themselves.
Cookbook attributes are saved inside the attributes directory, by default there is one file there called default.rb. In our example cookbook it contains two attributes:

    default['packages'] = []
    default['packages_default_action'] = 'install'

These are the default values defined in the cookbook. An empty array list of packages and the default action of the cookbook.

------------------------------------------------

## A chef runlist

# Vagrant-berkshelf

# Vagrant

#Questions?
