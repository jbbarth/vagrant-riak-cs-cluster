# -*- mode: ruby -*-
# vi: set ft=ruby :

$LOAD_PATH.unshift("lib") unless $LOAD_PATH.include?("lib")

#require "berkshelf/vagrant"
require "erlang_template_helper"

CENTOS = {
  sudo_group: "wheel",
  box: "opscode-centos-6.3",
  url: "https://opscode-vm.s3.amazonaws.com/vagrant/opscode_centos-6.3_chef-11.2.0.box"
}
UBUNTU = {
  sudo_group: "sudo",
  box: "opscode-ubuntu-12.04",
  url: "https://opscode-vm.s3.amazonaws.com/vagrant/opscode_ubuntu-12.04_chef-11.2.0.box"
}

NODES         = 1
OS            = UBUNTU
BASE_IP       = "33.33.33"
IP_INCREMENT  = 10

Vagrant.configure("2") do |cluster|
  (1..NODES).each do |index|
    last_octet = index * IP_INCREMENT

    cluster.vm.define "riak#{index}".to_sym do |config|
      # Configure the VM and operating system.
      config.vm.box = OS[:box]
      config.vm.box_url = OS[:url]
      config.vm.provider(:virtualbox) { |v| v.customize ["modifyvm", :id, "--memory", 1024] }

      # Setup the network and additional file shares.
      if index == 1
        config.vm.network :forwarded_port, guest: 8080, host: 8080
        config.vm.network :forwarded_port, guest: 8085, host: 8085
        config.vm.network :forwarded_port, guest: 8087, host: 8087
      end

      config.vm.hostname = "riak#{index}"
      config.vm.network :private_network, ip: "#{BASE_IP}.#{last_octet}"
      config.vm.synced_folder "lib/", "/tmp/vagrant-chef-1/lib"

      # Provision using Chef.
      config.vm.provision :chef_solo do |chef|
        chef.roles_path = "roles"

        if config.vm.box =~ /ubuntu/
          chef.add_recipe "apt"
        else
          chef.add_recipe "yum"
          chef.add_recipe "yum::epel"
        end

        chef.add_role "base"
        chef.add_role "riak"
        chef.add_role "stanchion" if index == 1
        chef.add_role "riak_cs"

        chef.json = {
          "authorization" => {
            "sudo" => {
              "groups" => [ OS[:sudo_group] ],
            }
          },
          "riak" => {
            "args" => {
              "+S" => 1,
              "-name" => "riak@33.33.33.#{last_octet}"
            }
          },
          "riak_cs" => {
            "args" => {
              "+S" => 1,
              "-name" => "riak-cs@33.33.33.#{last_octet}"
            },
            "config" => {
              "riak_cs" => {
                "anonymous_user_creation" => (ENV["RIAK_CS_CREATE_ADMIN_USER"].nil? ? false : true),
                "admin_key" => (ENV["RIAK_CS_ADMIN_KEY"].nil? ? "admin-key" : ENV["RIAK_CS_ADMIN_KEY"]).to_erl_string,
                "admin_secret" => (ENV["RIAK_CS_SECRET_KEY"].nil? ? "admin-secret" : ENV["RIAK_CS_SECRET_KEY"]).to_erl_string
              }
            }
          },
          "stanchion" => {
            "args" => {
              "+S" => 1,
              "-name" => "stanchion@33.33.33.#{last_octet}"
            },
            "config" => {
              "stanchion" => {
                "admin_key" => (ENV["RIAK_CS_ADMIN_KEY"].nil? ? "admin-key" : ENV["RIAK_CS_ADMIN_KEY"]).to_erl_string,
                "admin_secret" => (ENV["RIAK_CS_SECRET_KEY"].nil? ? "admin-secret" : ENV["RIAK_CS_SECRET_KEY"]).to_erl_string
              }
            }
          }
        }
      end
    end
  end
end
