# -*- mode: ruby -*-
# vi: set ft=ruby :
require "yaml"

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  vagrantfile_path = File.expand_path(File.dirname(__FILE__))
  env_definition_path = File.join(vagrantfile_path, "envs.yml")
  env_definitions = YAML.load_file(env_definition_path)

  env_definitions.each do |env_name, env|
    config.vm.define env_name do |machine|
      machine.vm.boot_timeout = 120
      machine.vm.box = env[:box]
      machine.vm.hostname = env_name
      machine.vm.provider "virtualbox" do |vbox|
        vbox.gui = false
        vbox.name = env_name
        vbox.memory = env[:memory]
        vbox.cpus = env[:cpus]
        # ensure time sync is on
        vbox.customize [
          "setextradata",
          :id,
          "VBoxInternal/Devices/VMMDev/0/Config/GetHostTimeDisabled",
          "0"
        ]
        # set sync threshold
        vbox.customize [
          "guestproperty",
          "set",
          :id,
          "/VirtualBox/GuestAdd/VBoxService/-timesync-set-threshold",
          15000
        ]
      end

      # ssh
      config.ssh.forward_agent = true

      # networking
			machine.vm.network :private_network, type: "dhcp"

			# folders
      [
				[".", "/vagrant"],
				["./shared", "/home/#{env_name}/shared"],
				["./envs/#{env_name}", "/home/#{env_name}/env"]
      ].each do |(host, guest)|
				unless File.exist?(File.expand_path(File.join(vagrantfile_path, host)))
					next puts "[DEVENV] skip nonexistent '#{host}'"
				end

        machine.vm.synced_folder host, guest, type: "nfs"
			end

			Array(env[:synced_folders]).each do |synced_folder|
				unless File.exist?(File.expand_path(synced_folder[:host]))
					next puts "[DEVENV] skip nonexistent '#{synced_folder[:host]}'"
				end
        machine.vm.synced_folder synced_folder[:host], synced_folder[:guest], type: "nfs"
			end

			# ports
			Array(env[:ports]).each do |port|
				machine.vm.network :forwarded_port, host: port[:host], guest: port[:guest]
			end

			machine.vm.provision :chef_zero do |chef|
				chef.cookbooks_path = ["cookbooks"]
        chef.nodes_path = ["nodes"]
        chef.add_recipe "devenv::#{env_name}"
			end
		end
	end

  # provisioning
  config.omnibus.chef_version = :latest
  config.berkshelf.enabled = true
  config.berkshelf.berksfile_path = "cookbooks/devenv/Berksfile"
end
