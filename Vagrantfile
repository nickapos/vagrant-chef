# -*- mode: ruby -*-"
# vi: set ft=ruby :

box_type  = "chef/centos-6.6"

Vagrant.configure("2") do |config|
  config.vm.box = "#{box_type}"
  config.berkshelf.enabled = true

  # ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  # Add user list
  # ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  config.vm.provision "chef_solo" do |chef|
  chef.json = {
      "packages" => ['git','openssl' ]
    }
    chef.add_recipe "packages::default"
  end

end
