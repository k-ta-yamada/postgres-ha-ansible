# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
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
    end
  end

  config.vm.provision "selinux:disabled", type: :shell do |s|
    s.inline = <<-SHELL
      sed -i 's/^SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
      setenforce 0
    SHELL
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

  # optional
  if Vagrant.has_plugin?("vagrant-proxyconf")
    config.proxy.http     = ENV["http_proxy"]
    config.proxy.https    = ENV["https_proxy"]
    config.proxy.no_proxy = ENV["no_proxy"]
  end

  # optional
  # if Vagrant.has_plugin?("vagrant-hostmanager")
  #   config.hostmanager.enabled      = true
  #   config.hostmanager.manage_host  = true
  #   config.hostmanager.manage_guest = true
  # end
end
