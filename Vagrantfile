# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :raid => {
        :box_name => "generic/ubuntu2204",
        :vm_name => "raid",
        :net => [
           ["192.168.11.150",  2, "255.255.255.0", "mynet"],
        ],
        :disks => {
          :disk1 => { size: "5120", port: 1 },
          :disk2 => { size: "5120", port: 2 },
          :disk3 => { size: "5120", port: 3 },
          :disk4 => { size: "5120", port: 4 },
          :disk5 => { size: "5120", port: 5 },
        }
  }
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

    config.vm.define boxname do |box|
   
      box.vm.box = boxconfig[:box_name]
      box.vm.host_name = boxconfig[:vm_name]
      
      box.vm.provider "virtualbox" do |v|
        v.memory = 768
        v.cpus = 1
        
        if boxconfig.key?(:disks)
          boxconfig[:disks].each do |name, disk|
            v.customize [
            ]
            v.customize [
              "storageattach", :id,
              "--storagectl", "SATA Controller",
              "--port", disk[:port].to_s,
              "--device", "0",
              "--type", "hdd",
              "--medium", "#{name}.vdi"
            ]
          end
        end
      end
     
      
      boxconfig[:net].each do |ipconf|
        box.vm.network("private_network", ip: ipconf[0], adapter: ipconf[1], netmask: ipconf[2], virtualbox__intnet: ipconf[3])
      end

      if boxconfig.key?(:public)
        box.vm.network "public_network", boxconfig[:public]
      end
      box.vm.provision "shell", inline: <<-SHELL
        mkdir -p ~root/.ssh
        cp ~vagrant/.ssh/auth* ~root/.ssh
        sudo sed -i 's/\#PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
        systemctl restart sshd
      SHELL
    end
  end
end
