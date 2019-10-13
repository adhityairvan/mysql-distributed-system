# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.

Vagrant.configure("2") do |config|
  
  # MySQL Cluster dengan 3 node
  (1..3).each do |i|
    config.vm.define "mysql#{i}" do |node|
      node.vm.hostname = "mysql#{i}"
      node.vm.box = "bento/ubuntu-16.04"
      node.vm.network "private_network", ip: "10.1.15.14#{i+2}"
      
      node.vm.provider "virtualbox" do |vb|
        vb.name = "mysql#{i}"
        vb.gui = false
        vb.memory = "512"
      end
  
      node.vm.provision "shell", path: "provision.sh", privileged: false
    end
  end

  config.vm.define "proxy" do |proxy|
    proxy.vm.hostname = "proxy"
    proxy.vm.box = "bento/ubuntu-16.04"
    proxy.vm.network "private_network", ip: "10.1.15.146"
    proxy.vm.network "private_network", ip: "10.10.15.146"
    
    proxy.vm.provider "virtualbox" do |vb|
      vb.name = "proxy"
      vb.gui = false
      vb.memory = "256"
    end

    proxy.vm.provision "shell", path: "provisionProxy.sh", privileged: false
  end

  config.vm.define "webserver" do |webserver|
    webserver.vm.hostname = "webserver"
    webserver.vm.box = "bento/ubuntu-16.04"
    webserver.vm.network "private_network", ip: "10.10.15.147"
    config.vm.network "forwarded_port", guest: 80, host: 8080
    config.vm.synced_folder ".", "/vagrant", mount_options: ["dmode=755,fmode=755"]

    webserver.vm.provider "virtualbox" do |vb|
      vb.name = "webserver"
      vb.gui = false
      vb.memory = "512"
    end
    webserver.vm.provision "shell", path: "provisionWebserver.sh", privileged: false

  end
  
end
