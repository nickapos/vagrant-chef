# -*- mode: ruby -*-"
# vi: set ft=ruby :

box_type  = "bento/centos-7.2"

Vagrant.configure("2") do |config|
  config.vm.box = "#{box_type}"
  config.berkshelf.enabled = true

  # ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  # Add packages
  # ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  config.vm.provision "chef_solo" do |chef|
  chef.json = {
      "packages" => ['git','screen','openssl' ]
    }
    chef.add_recipe "packages::default"
  end

end
