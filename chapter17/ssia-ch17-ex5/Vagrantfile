# -*- mode: ruby -*-
# vi: set ft=ruby :
userpath = "#{ENV['HOMEPATH']}"

Vagrant.configure("2") do |config|
  # if (/cygwin|mswin|mingw|bccwin|wince|emx/ =~ RUBY_PLATFORM) != nil
  #   config.vm.synced_folder "~/.m2", "/home/vagrant/.m2", mount_options: ["dmode=700,fmode=600"]
  # else
  #   config.vm.synced_folder "#{userpath}/.m2", "/home/vagrant/.m2"
  # end
  config.vm.define "cd" do |d|
    d.vm.box = "ubuntu/focal64"
    d.vm.network "private_network", ip: "10.100.98.200" 
    d.vm.provider "virtualbox" do |v|
	  v.memory = 3096
	  v.cpus = 2
    end
    d.vm.provision "shell", path: "./scripts/provision-dev-vm.sh"
  end
end