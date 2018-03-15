# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "centos/7"
  config.vm.box_version = "1710.01"


  hosts = [
    { hostname: :pg1, ip: %w[192.168.1.201 192.168.2.201] },
    { hostname: :pg2, ip: %w[192.168.1.202 192.168.2.202] }
  ]
  controle_node = { hostname: :pg0, ip: %w[192.168.1.200 192.168.2.200] }
  hosts.unshift(controle_node) if /cygwin|mswin|mingw|bccwin|wince|emx/ =~ RUBY_PLATFORM

  hosts.each do |host|
    config.vm.define host[:hostname] do |node|
      node.vm.provider "virtualbox" do |v|
        v.linked_clone = true
      end

      node.vm.hostname = host[:hostname]

      host[:ip].each { |ip| node.vm.network "private_network", ip: ip }

      node.vm.provision "ssh:root", type: :shell do |s|
        private_key = "/vagrant/.vagrant/machines/#{node.vm.hostname}/virtualbox/private_key"
        s.inline = <<-SHELL
          sudo su - root -c "mkdir -p /root/.ssh/"
          sudo chmod 700 /root/.ssh/
          ssh-keygen -yf #{private_key} > /root/.ssh/authorized_keys
          sudo chmod 600 /root/.ssh/authorized_keys
        SHELL
      end
    end
  end

  config.vm.provision "selinux:disabled", type: :shell do |s|
    s.inline = "sed -i 's/^SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config"
  end

  config.vm.provision "sshd:password_authentication", type: :shell do |s|
    s.inline = <<-SHELL
      sed -i 's/^PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
      systemctl reload sshd.service
    SHELL
  end

  config.vm.provision "timedatectl:set-timezone:Asia/Tokyo", type: :shell do |s|
    s.inline = "timedatectl set-timezone Asia/Tokyo"
  end

  config.vm.provision "yum:install:tmux", type: :shell do |s|
    s.inline = "yum install tmux -y"
  end

  config.vm.provision "yum:install:vim", type: :shell do |s|
    s.inline = "yum install vim-enhanced -y"
  end


  if Vagrant.has_plugin?("vagrant-vbguest")
    # puts "-- has_plugin: vagrant-vbguest"
    config.vm.synced_folder ".", "/vagrant", type: "virtualbox"
  end

  if Vagrant.has_plugin?("vagrant-proxyconf")
    # puts "-- has_plugin: vagrant-proxyconf"
    # puts "   http_proxy  is [#{ENV["http_proxy"]}]"
    # puts "   https_proxy is [#{ENV["https_proxy"]}]"
    # puts "   no_proxy    is [#{ENV["no_proxy"]}]"
    config.proxy.http     = ENV["http_proxy"]
    config.proxy.https    = ENV["https_proxy"]
    config.proxy.no_proxy = ENV["no_proxy"]
  end

  if Vagrant.has_plugin?("vagrant-hostmanager")
    # config.vm.provision :hostmanager
    config.hostmanager.enabled      = true
    config.hostmanager.manage_host  = true
    config.hostmanager.manage_guest = true
  end

  # config.vm.provider "virtualbox" do |v|
  #   v.linked_clone = true
  # end

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.0.100"
  # config.vm.network "private_network", ip: "192.168.1.100"
  # config.vm.network "private_network", ip: "192.168.2.100"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL
end
