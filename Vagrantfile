# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true

  # VMs to be monitored by prometheus
  (1..3).each do |i|
    config.vm.define "sg-node-#{i}" do |node|
      node.vm.box = "bento/ubuntu-20.04"
      node.vm.network "private_network", ip: "192.168.50.8#{i}"
      node.vm.hostname = "sg-node-#{i}"
      node.vm.provision :ansible do |ansible|
        ansible.playbook = "ansible/provision.yaml"
        ansible.inventory_path = "ansible/inventory"
        ansible.become = true
      end

      node.vm.provider :virtualbox do |vb|
        vb.memory = "512"
        vb.cpus = "1"
      end
    end
  end

  # monitoring stack node
  config.vm.define "sg-app" do |app|
    app.vm.box = "bento/ubuntu-20.04"
    app.vm.hostname = "sg-app"
    app.vm.network "private_network", ip: "192.168.50.88"

    app.vm.provision :ansible do |ansible|
      ansible.playbook = "ansible/provision.yaml"
      ansible.inventory_path = "ansible/inventory"
      ansible.become = true
    end

    app.vm.provider :virtualbox do |vb|
      vb.memory = "4096"
      vb.cpus = "4"
    end
  end
end
