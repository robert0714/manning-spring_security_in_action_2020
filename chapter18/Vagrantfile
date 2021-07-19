# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.define "cd" do |d|
    d.vm.box = "ubuntu/xenial64"
    d.vm.network "private_network", ip: "10.100.98.200"
    #d.vm.network "forwarded_port", guest: 9000, host: 9000
    #d.vm.network "forwarded_port", guest: 9092, host: 9092
    d.vm.provider "virtualbox" do |v|
	  v.memory = 3096
	  v.cpus = 2
    end
    d.vm.provision "shell", path: "./scripts/provision-dev-vm.sh"
  end
end