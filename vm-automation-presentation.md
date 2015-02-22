% Automated vm generation
% Nick Apostolakis
% 27th of February 2015

# A walkthrough of the technologies used in NCR Edinburgh

The goal is to create a fully configured vm artifact ready to be imported in the virtualization software of choice. We will use VirtualBox for the development system

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
  * The run_list

## Cookbook attributes

One important feature of attributes is that they can be over-riden during runtime, both from the calling scripts and the cookbooks themselves.
Cookbook attributes are saved inside the attributes directory, by default there is one file there called default.rb. In our example cookbook it contains two attributes:

    default['packages'] = []
    default['packages_default_action'] = 'install'

These are the default values defined in the cookbook. An empty array list of packages and the default action of the cookbook.

------------------------------------------------

## A chef run_list

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

If the Vagrant Berkshelf plugin is installed, it will intelligently detect when a Berksfile is present in the same working directory as the Vagrantfile.

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


#Questions?
