---
title:Automated vm generation
author: Nick Apostolakis
date: 27th of February 2015
---

# Introduction

As virtual machines are becoming increasingly important by the day both for 
organizations, we will atempt here to provide a brief introductions of the tools
used in NCR Edinburgh for the automation and provisioning of our infrastructure.

These tools can be used either in a corporate environment in order to provide 
infrastructure standarization and automation both for development and production
systems.
All of the tools mentioned here are open source projects and can be downloaded at
no cost.


# A walkthrough of the technologies used in NCR Edinburgh

The goal is to create a fully configured vm artifact ready to be imported in the virtualization software of choice. We will use VirtualBox for the development system.Our technology stack is the following:

  * _[Packer](https://www.packer.io/)_ used for the creation of custom Vagrant boxes
  * _[Berkshelf](https://downloads.chef.io/chef-dk/)_ part of Chef Development toolkit. Used for cookbook dependency management
  * _[Chef-solo](https://downloads.chef.io/chef-dk/)_
  * _[Vagrant-berkshelf](https://github.com/berkshelf/vagrant-berkshelf)_ used to transfer the cookbooks on the new vm
  * _[Vagrant](https://www.vagrantup.com/)_ combines all the aforementioned technologies to create and provision the new vm

# Packer

Packer is an open source tool for creating identical machine images for multiple platforms from a single source configuration. 

Packer does not replace configuration management like Chef or Puppet. In fact, when building images, Packer is able to use tools like Chef or Puppet to install software onto the image.

A machine image is a single static unit that contains a pre-configured operating system and installed software which is used to quickly create new running machines. Machine image formats change for each platform. Some examples include AMIs for EC2, VMDK/VMX files for VMware, OVF exports for VirtualBox, etc.

Although nowadays Packer supports Chef itself, we are basically using its Vagrant post-processor to create custom Vagrant boxes from the net installation iso of Centos 6.6. Originally Packer did not support configuration management systems, so we used Vagrant instead. Nowadays its feature list is quite close to the Vagrant

# Packer template file example

  * Templates are JSON files that configure the various components of Packer in order to create one or more machine images. 
  * Templates are given to commands such as packer build, which will take the template and actually run the builds within it, producing any resulting machine images.

Packer templates are often brittle and not so easy to use as Vagrant files. The following example retrieves the ubuntu server iso and prepares a system using shell as a provisioner and executing the script setup_things.sh

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
  * is a tool that can be used to manage and define cookbooks dependencies.
  * is used by our vm creation tool Vagrant to download and install all the cookbooks that we need on the new vm
  * it is part of Chef Development Kit (ChefDK)
  * The main configuration file of Berkshelf is a file called Berksfile

## Berkshelf commands

    berks 

will download the cookbooks and 

    berks vendor

will upload them to the new vm. You dont need to do this manually, there is a Vagrant plugin that takes care of everything

--------------------------------------

## Berksfile


The Berksfile defines the behaviour of the berkshelf tool. There is a community cookbook repository called chef supermarket and Berkshelf can acces them by using the following syntax:

    source "https://supermarket.chef.io"
    cookbook 'cookbook_name', "~> 2.6"


Berkhelf can also use cookbooks from alternative locations (e.g git repos, github, gitlab etc) with a small change in the Berksfile syntax:

    cookbook 'packages', git: 'git@github.com:mattray/packages-cookbook.git'

# Chef

Chef is one of the most prominent configuration managment and vm provisioning 
opensource systems. It is mainly consisted by a nummber of execution blocks called
resoources, bundled together in files called recipes. A collection of recipes that can
be combined to preform a specific task are often combined in larger packages 
called cookbooks. For more details about the internal structure of chef, please
refer to the first part of the vm automation.

For the purposes of this presentation we are going to use a very simple
cookbook, that takes a list of packages and installs it on the new vm.
It is developed by a Chef employee Matt Ray and is distributed unde the
Apache License. It does not matter which distribution we choose to use,
it will work on all of them, Chef takes care of the differences in
package managers. By using a tool like Chef, we can ignore the underlying 
differences in package managers, repositories and initialization scripts
and focus at the job in hand. Chef allow us to describe very comples structures
in a declarative way.


##Core chef concepts:
  
  * Cookbook attributes
  * The run_list

### Cookbook attributes

One important feature of attributes is that they can be over-riden during runtime, both from the calling scripts and the cookbooks themselves.
Cookbook attributes are saved inside the attributes directory, by default there is one file there called default.rb. In our example cookbook it contains two attributes:

    default['packages'] = []
    default['packages_default_action'] = 'install'

These are the default values defined in the cookbook. An empty array list of packages and the default action of the cookbook.

------------------------------------------------

### A chef run_list

A run-list is:

  * An ordered list of roles and/or recipes that are run in an exact order; if a recipe appears more than once in the run-list, the chef-client will never run that recipe twice
  * Always specific to the node on which it runs, though it is possible for many nodes to have run-lists that are similar or even identical 
  * Stored as part of the node object on the Chef server
  * Maintained using knife and uploaded to the Chef server or via the Chef management console user interface

In the chef-solo context, a run_list is limited to a specific chef-solo run and isolated on the vm it is running, it does not require any external chef resources (e.g chef server)

Plainly put, a run_list is the list of the recipes we want to execute from one or more cookbooks. These cookbooks need to be available and accessible on a predefined location.

It looks like this: 

    { "run_list": ["recipe[packages::default]", "recipe[anotherthing]"] }

# Vagrant-berkshelf

Vagrant Berkshelf is a Vagrant plugin that adds Berkshelf integration to the Chef provisioners. Vagrant Berkshelf will automatically download and install cookbooks onto the Vagrant Virtual Machine. 

If the Vagrant Berkshelf plugin is installed, it will detect when a Berksfile is present in the same working directory as the Vagrantfile.

Vagrant-berkshelf is installed using the following command:

    vagrant plugin install vagrant-berkshelf

# Vagrant

Vagrant is a tool for building complete development environments. With an easy-to-use workflow and focus on automation. 

It uses pre packaged templates called boxes as a starting point and is using an open extendable interface in order to provide more features in the form of plugins. Vagrant-berkshelf is one of those plugins, and that makes work with chef a lot easier. 

There are a lot of freely available community vagrant boxes for any platform imaginable. In NCR Edinburgh we build our own from the official CentOS sources.

Vagrant supports out of the box a number of provisioners, among themselves chef and salt. In the following example we are going to use the chef provisioner.

It also supports a number of virtualisation providers, with VirtualBox as a default and VMWare as the suggested platform for production environments. All these providers are distributed as vagrant plugins

Vagrant is controlled by a single file called Vagrantfile

  * Example Vagrantfiles can be found [here](https://github.com/patrickdlee/vagrant-examples)

------------------------------------------------------------

## The most important vagrant commands

  * _vagrant plugin install plugin-name_ (installs vagrant plugins)
  * _vagrant up_ (imports the base box in the local vagrant repo, starts and provisions the new vm)
  * _vagrant provision_ (runs the provisioner blocks)
  * _vagrant ssh_ (logs into the new vm)
  * _vagrant reload_ (restarts vagrant machine, loads new Vagrantfile configuration)
  * _vagrant halt_ (stops the vagrant machine)
  * _vagrant destroy_ (stops and deletes all the files of the vagrant machine)


All the vagrant vms appear as VirtualBox vms, so you can use VirtualBox interface to export a vm to and ova

--------------------------------------------------------------
## Vagrantfile

A Vagrant file looks like this:

    # -*- mode: ruby -*-"
    # vi: set ft=ruby :

    box_type  = "chef/centos-6.6"

    Vagrant.configure("2") do |config|
      config.vm.box = "#{box_type}"
      config.berkshelf.enabled = true

      # ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
      # Add packages
      # ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
      config.vm.provision "chef_solo" do |chef|
        chef.json = {
          "packages" => ['git', 'screen', 'vim','openssl' ]
        }
        chef.add_recipe "packages::default"
      end

    end

We can see here, that the Vagrantfile syntax is actually a DSL (Domain Specific
Language) it allows us to use a relatively simple, human readable, declarative
syntax to define the various attributes we want our new vm to have.

In this file, we are defining the box type, we are using one of the community 
vagrant boxes provided by opscode the company that creates chef, we are enabling
the vagrant berkshelf support since we want to use berkshelf to handle the 
cookbook management for us and use the chef provisioner in order to call the 
default recipe of the packages cookbook.

Vagrant will override the default attribute with name packages and provide to 
chef-solo a list of packages that need either to be installed or removed.
The default action (also defined by an attribute) is install, so these files will
be installed. As we can see we do not define anywhere how this is going to be done.
The chef run is completely declarative. All the complexities related to package
managers, packages and repos are handled under the hood by chef.


# The packages cookbook recipe

The cookbook recipe we are executing with this vagrant file goes through the following operations:

    Chef::Log.info "packages:#{node['packages'].inspect}"
    
    if node['packages'].is_a?(Array)
      node['packages'].each do |pkg|
        package pkg do
          action node['packages_default_action'].to_sym
        end
      end
    elsif node['packages'].is_a?(Hash)
      node['packages'].each do |pkg, act|
        package pkg.to_s do
          action act.to_sym
        end
      end
    else
      Chef::Log.warn('`node["packages"]` must be an Array or Hash.')
    end


Chef recipes are also a ruby DSL, and even though it is not pure ruby they are 
a bit more technical than Vagrant directives. Despite that, the recipes are also 
declarative, and as we can see in this example, we are using the resource package
in order to define the concept of a package and its installation action and not
the exact package name and the name of the package manager.

This recipe will check if the provided attribute is a hash or an array, and act
accordingly. Ruby is a dynamically typed language so there is no way to infer the
type of the data types at the beginning of the execution.

After the installation of the packages the chef run terminates.

# Vagrant or Packer

A reasonable question is why do we use two tools that are doing a similar job.
Packer and Vagrant are developed by the same company. Originally Packer did
not support all the external provisioners it supports today, so its use was mostly
limited in creating vagrant boxes. Today it supports most of the tools Vagrant
supports and can be used instead of Vagrant.

However there are a few points that you may want to consider before using Packer
for everything. 

* Packer uses a configuration file that is written in json, and that makes it more difficult to edit manually and more easy to corrupt. 
* Vagrant's DSL syntax is a lot easier to read and edit. 
* Packer is not as widely used and as extensible as Vagrant. 
* Vagrant has a plugin interface, and that makes it easy to create and distribute all kind of plugins that can add support for different virtualization systems, different provisioning systems, cookbook management utilities (berkshelf). If we were using Packer we would have to preform all these steps manually or re-invent the wheel and implement our own
tools to preform the steps automatically.


# Conclusion

At this poing we should have a live virtual machine created from scratch by our 
scripts. We can log in to this vm and use it with the command 

    vagrant login

or create a new vagrant box out of it that can be uploaded to the public vagrant 
repository of boxes.
The vm can also be exported to an ova, if we use the VirtualBox tools and be
distributed to other systems, or members of our organization.

This vm is completely disposable since it can be recreated at any moment through
 the use of the scripts either manually or through an automation system like 
 Jenkins.

