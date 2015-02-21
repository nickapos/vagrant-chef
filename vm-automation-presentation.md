% Automated vm generation
% Nick Apostolakis
% 27th of February 2015

# A walkthrough of the technologies used in NCR Edinburgh

The goal is to create a fully configured vm artifact ready to be imported in the virtualization software of choice. We will use VirtualBox for the development system

  * [Packer](https://www.packer.io/) This is used for the creation of custom Vagrant boxes
  * [Vagrant](https://www.vagrantup.com/) Vagrant will be used to generate the finished product
  * [Berkshelf](https://downloads.chef.io/chef-dk/) Part of Chef Development toolkit. Used for defining cookbook dependencies
  * Vagrant-berkshelf The vagrant backend that is used to transfer the cookbooks on the new vm

# Enter configuration and vm managment systems

## With the configuration management systems we can:

  * Define in a declarative way, the desired state of the a system or service. 
  * Distribute these informations in a consistent and safe way.
  * Achieve idepondence
  * Propagate changes in a consistent and safe way
  * service state consistency. (e.g automatic restart of failed services)


## With the vm management and orchestration tools we can:

  * define dependencies between vms
  * define types of vms
  * create and destroy whole environments of these vms
  * manage vms as groups or individualy
  * use various policies for rolling out vm upgrades
  * horizontal auto scaling of whole environments
  * automatic restart of failed vms


# Most prominent tools

## Configuration management systems

  * [cfengine](http://cfengine.com/)
  * [Puppet](http://puppetlabs.com/)
  * [Chef](https://www.chef.io/)
  * [Salt](http://saltstack.com/)
  * [Ansible](http://www.ansible.com/home)

## Vm management & orchestration systems

  * [Ganeti](https://code.google.com/p/ganeti/)
  * [Openstack](https://www.openstack.org/)
  * [AWS CloudFormation](http://aws.amazon.com/cloudformation/)
  * [Kubernetes](https://github.com/googlecloudplatform/kubernetes)
  * [Apache Brooklyn](https://brooklyn.incubator.apache.org/)

# Our focus today will be on the configuration management systems.

## All those systems share some common concepts.

  * They have a data harvesting service (factor, ohai). This tool retrieves varioys attributes from the system:
    - Platform
    - Operating system
    - Hostname
    - Ip
    - etc
  * They are defining resources in a declarative way, that can be used to manipulate the system. A resource can be:
    - a user
    - a directory
    - a service
    - a package
    - etc
  * Multiple mode of execution (not supported by all tools)
    - Standalone mode
    - Server/client mode

# In NCR Edinburgh we are using Opscode Chef
All the above, are supported by Chef. Chef gets its terminology from cooking.

Chef is using the resource definition, as described in the previous slide. 

* A collection of chef resources is called a recipe. 
* A collection of recipes is called a cookbook.
* the tool that is used for cookbook scaffolding and management is called knife
* the integration test suite for cookbooks is called test-kitchen

# A peek inside Chef

Chef uses a Ruby DSL for its recipe definition.
A simple resource call example:

    directory "/var/lib/foo" do
      owner 'root'
      group 'root'
      mode '0644'
      action :create
    end

# A cookbook structure

A cookbook is not made only to contain recipes. It is a complex package that can contain:

* files that will be copied on the new vm
* templates
* attributes
* tests
* new resources
* providers
* dependencies between cookbooks
* data bags

>A data bag is a global variable that is stored as JSON data and is accessible from a Chef server. A data bag is indexed for searching and can be loaded by a recipe or accessed during a search.
>A data bag item may be encrypted using shared secret encryption. This allows each data bag item to store confidential information (such as a database password) or to be managed in a source control system (without plain-text data appearing in revision history). Each data bag item may be encrypted individually; if a data bag contains multiple encrypted data bag items, these data bag items are not required to share the same encryption keys.

# Chef supports three execution modes

* Client/server
* Chef solo
* Chef zero

# Chef solo

* Chef solo is an isolated run of chef converge. 
* It can use recipes, attributes, templates and data bags but can not use the advanced chef features like search etc. 

>Chef solo is used when we want to converge an isolated vm, without having to connect to an external chef server. 
It is the approach used by all of our vms, although the implementation differs slightly between the ticketing and the connections vms.
The connections vms (connections & mcw) are using databags for the tomcat instances, so they can be extended without modifying the cookbook. As described in the previous section a databag is an external data source for a cookbook.

# Chef zero

* Chef zero is also called a poor mans chef server
* It is an in memory chef server
* It is a recent addition (after Chef v11.0)
* It supports all the Chef solo functionality
* Plus a few more advanced features, like data bag searches

# Chef server

* It is a client server architecture
* Chef-client is an agent installed on the vm that runs regularly and converges the vm against the specified role
* Chef server is a repository of almost all the chef related artifacts (cookbooks, roles, databags)
* Chef cookbooks can be versioned, chef roles can not.
* It offers a web interface through which the chef managed nodes 
* There is also a REST API that can be used to search and retrieve information about the chef artifacts

Chef server is used in our Jenkins cluster. Our Jenkins slaves have a quite complex setup, since we need to support consistently a lot of different technologies. 

# Which execution mode is the best?

* It depends on the environment and the architecture. 

>Chef solo/zero are suitable for isolated environments, where we have not full access on the network infrastructure, or where we can not change the architecture of the environment at will. Although it does not provide the full functionality this execution mode is quite powerful. Chef server, has complex setup so it is quite possible that the cost of setting up a chef server is more than the benefit of using it.

* It makes sense to set up a chef server, when we have a complex architecture and the power to shape the infrastructure at will. 

>This will allow us to manage a lot of different environments, and scale it horizontally with thousand of vms, with minimal effort. If the architecture is even more complex, we can set up chef servers that manage chef servers. Chef can be customized a great deal, by using custom event handlers and various parts of it, including ohai can be extended with new custom plugins. 


#Questions?
