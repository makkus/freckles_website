# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.box = "fujimakishouten/debian-stretch64"

  # config.vm.box_check_update = false

  config.vm.network "forwarded_port", guest: 8280, host: 8280
  config.vm.network "forwarded_port", guest: 8281, host: 8281
  config.vm.network "private_network", ip: "10.11.12.13"

  config.vm.synced_folder ".", "/vagrant", id: "vagrant"
  config.vm.synced_folder "/home/markus/projects/freckles", "/freckles", id: "freckles", owner: "vagrant", group: "vagrant"
  config.vm.synced_folder "/home/markus/projects/adapters", "/adapters", id: "adapters", owner: "vagrant", group: "vagrant"
  config.vm.synced_folder "/home/markus/projects/ansible", "/ansible", id: "ansible", owner: "vagrant", group: "vagrant"
  # config.vm.synced_folder "/home/markus/.local/inaugurate", "/home/vagrant/.local/inaugurate", id: "inaugurate", owner: "vagrant", group: "vagrant"

  config.vm.provider "virtualbox" do |vb|
    vb.gui = false
    vb.memory = "2048"
  end

  config.vm.provision "shell", privileged: false, inline: <<-SHELL
    # sudo ln -s /home/vagrant /home/markus
    # sudo chown vagrant:vagrant /home/vagrant/.local

    # wget -O - https://freckles.io | sudo bash -s -- freckelize -r gh:makkus/frecklets grav -f /vagrant/ --port 8280 --nginx-user vagrant
  SHELL
end
