# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
    :pam => {
        :box_name => "almalinux/9",
        :cpus => 2,
        :memory => 1024,
        :ip_addr => '192.168.50.10',
    },
}

Vagrant.configure("2") do |config|
  MACHINES.each do |boxname, boxconfig|
      config.vm.synced_folder ".", "/vagrant", disabled: true
      config.vm.define boxname do |box|
          box.vm.box = boxconfig[:box_name]
          box.vm.host_name = boxname.to_s
          box.vm.network "private_network", ip: boxconfig[:ip_addr]
          box.vm.provider :virtualbox do |vb|
            vb.name = boxname.to_s
            vb.memory = boxconfig[:memory]
            vb.cpus = boxconfig[:cpus]
          end
          box.vm.provision "shell", inline: <<-SHELL
          sed -i 's/^PasswordAuthentication.*$/PasswordAuthentication yes/' /etc/ssh/sshd_config
          systemctl restart sshd.service
          SHELL
      end
  end
end
