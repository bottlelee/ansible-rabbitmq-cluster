# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV["LC_ALL"] = "en_US.UTF-8"

Vagrant.require_version ">= 2.0.0"

$vm_box = "ubuntu/xenial64"
# $vm_box = "centos/7"
$instances = 3
$apt_proxy = "http://192.168.205.16:3142"

Vagrant.configure("2") do |config|
  # always use Vagrants insecure key
  config.ssh.insert_key = false
  config.vm.box_check_update = false
  config.vm.box = $vm_box
  config.vm.synced_folder ".", "/vagrant", disabled: true

  if Vagrant.has_plugin?("vagrant-vbguest") then
    config.vbguest.auto_update = false
  end

  if Vagrant.has_plugin?("vagrant-proxyconf") and $vm_box == "ubuntu/xenial64" then
    config.apt_proxy.http = $apt_proxy || ""
    config.apt_proxy.https = "DIRECT"
  end

  if Vagrant.has_plugin?("vagrant-proxyconf")
    config.proxy.http     = ENV['HTTP_PROXY'] || ENV['http_proxy'] || ""
    config.proxy.https    = ENV['HTTPS_PROXY'] || ENV['https_proxy'] ||  ""
    config.proxy.no_proxy = $no_proxy
  end

  (1..$instances).each do |instance_id|
    $vm_name = "rabbitmq-#{instance_id.to_s.rjust(2, '0')}"

    config.vm.define vm_name = $vm_name do |config|
      config.vm.hostname = vm_name
      config.vm.network "private_network", ip: "172.28.128.1#{instance_id.to_s.rjust(2, '0')}"
      config.vm.network "forwarded_port", guest: 5672, host: 5672,
        auto_correct: true

      config.vm.provider "virtualbox" do |vb|
        vb.customize ["modifyvm", :id, "--natdnshostresolver1", "off"]
        vb.customize ["modifyvm", :id, "--natdnsproxy1", "off"]
        vb.name = vm_name
        vb.memory = "2048"
        vb.cpus = "2"
      end

      if instance_id == $instances
        config.vm.provision "ansible" do |ansible|
          ansible.limit = "all"
          ansible.playbook = "play-all.yml"
        end
      end
    end
  end
end
