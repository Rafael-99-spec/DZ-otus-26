# -*- mode: ruby -*-
# vim: set ft=ruby :
# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {

  :pxeserver => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.7.1', adapter: 2, netmask: "255.255.255.0", virtualbox__intnet: "pxenet"},
                   #{ip: '192.168.0.1', adapter: 3, netmask: "255.255.255.240", virtualbox__intnet: "dirrectors"},
                ]
  },
  
  :pxeclient => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.7.2', adapter: 2, netmask: "255.255.255.0", virtualbox__intnet: "pxenet"}
                ]
  },

}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

    config.vm.define boxname do |box|

        box.vm.box = boxconfig[:box_name]
        box.vm.host_name = boxname.to_s

        boxconfig[:net].each do |ipconf|
          box.vm.network "private_network", ipconf
        end
        
        if boxconfig.key?(:public)
          box.vm.network "public_network", boxconfig[:public]
        end

        box.vm.provision "shell", inline: <<-SHELL
          mkdir -p ~root/.ssh
                cp ~vagrant/.ssh/auth* ~root/.ssh
        SHELL
        
        case boxname.to_s
        when "pxeserver"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
              yum update -y
              yum install net-tools tcpdump -y
              echo "toor" | sudo passwd root --stdin
            SHELL
        when "pxeclient"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
              yum update -y
              yum install net-tools tcpdump -y
              echo "toor" | sudo passwd root --stdin
            SHELL
        end

      end

  end
  
  
end
