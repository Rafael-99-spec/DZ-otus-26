# -*- mode: ruby -*-
# vim: set ft=ruby :
# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
:"web-client" => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.11.102', adapter: 2, netmask: "255.255.255.0", virtualbox__intnet: "net2"},
                ]
  },
  :"dhcp-server" => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.11.101', adapter: 2, netmask: "255.255.255.0", virtualbox__intnet: "net2"},
                ]
  }
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
        when "web-client"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            yum install epel-release -y
            yum install wget nano httpd net-tools, wget -y
            systemctl start httpd
            systemctl enable httpd
            wget https://mirror.yandex.ru/centos/8.2.2004/isos/x86_64/CentOS-8.2.2004-x86_64-minimal.iso
            mkdir /mnt/centos8-install/
            mount -o loop,ro -t iso9660 CentOS-8.2.2004-x86_64-minimal.iso /mnt/centos8-install/
            cp -r /mnt/centos8-install/ /var/www/html/
            cp /vagrant/ks.cfg /var/www/html/centos8-install/
            systemctl restart httpd.service
            chmod -R 755 /var/www/html/centos8-install/

          SHELL
          when "dhcp-server"
            box.vm.provision "shell", run: "always", inline: <<-SHELL
            yum install epel-release -y
            yum install wget nano net-tools xinetd dracut-network tftp-server dhcp -y
            cp /vagrant/tftp /etc/xinetd.d/
            cp /vagrant/dhcpd.conf /etc/dhcp/
            mkdir -p /etc/dhcp/rsv.d
            mkdir -p /etc/dhcp/subnet.d
            cp /vagrant/deploy-hosts.rsv /etc/dhcp/rsv.d/           
            cp /vagrant/subnet.conf /etc/dhcp/subnet.d/
            mkdir -p /var/lib/tftpboot/pxelinux.cfg
            chmod 755 /var/lib/tftpboot/pxelinux.cfg
            wget http://192.168.11.102/centos8-install/BaseOS/Packages/syslinux-tftpboot-6.04-4.el8.noarch.rpm
            rpm2cpio /home/vagrant/syslinux-tftpboot-6.04-4.el8.noarch.rpm | cpio -dimv
            cp /home/vagrant/tftpboot/ldlinux.c32 /home/vagrant/tftpboot/libutil.c32 /home/vagrant/tftpboot/menu.c32 /home/vagrant/tftpboot/pxelinux.0 /var/lib/tftpboot/
            wget http://192.168.11.102/centos8-install/images/pxeboot/initrd.img
            wget http://192.168.11.102/centos8-install/images/pxeboot/vmlinuz
            cp /home/vagrant/initrd.img /home/vagrant/vmlinuz /var/lib/tftpboot/
            cp /vagrant/default /var/lib/tftpboot/pxelinux.cfg/ 
            systemctl start dhcpd
            systemctl enable dhcpd
            systemctl start tftp
            systemctl enable tftp
            systemctl start xinetd
            systemctl enable xinetd
           SHELL
        end

    end

  end
  
  
end
