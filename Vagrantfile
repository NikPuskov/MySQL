ENV['VAGRANT_SERVER_URL'] = 'https://vagrant.elab.pro'

MACHINES = {
  :master => {
        :box_name => "centos/7",
        :ip_addr => '192.168.56.150'
  },
  :slave => {
        :box_name => "centos/7",
        :ip_addr => '192.168.56.151'
  }
}

Vagrant.configure("2") do |config|

  MACHINES.each_with_index do |(boxname, boxconfig), index|

      config.vm.define boxname do |box|

          box.vm.box = boxconfig[:box_name]
          box.vm.host_name = boxname.to_s

          box.vm.network "private_network", ip: boxconfig[:ip_addr]

          box.vm.provider :virtualbox do |vb|
            vb.customize ["modifyvm", :id, "--memory", "1024"]  
          end
          
          config.vm.provision "shell", inline: <<-SHELL
                    sudo sed -i s/mirror.centos.org/vault.centos.org/g /etc/yum.repos.d/*.repo
                    sudo sed -i s/^#.*baseurl=http/baseurl=http/g /etc/yum.repos.d/*.repo
                    sudo sed -i s/^mirrorlist=http/#mirrorlist=http/g /etc/yum.repos.d/*.repo
          SHELL
          
          if index == MACHINES.size - 1
            box.vm.provision "playbook1", type:'ansible' do |ansible|
              ansible.playbook = "ansible/provision.yml"
              ansible.inventory_path = "ansible/inventory.yml"              
              ansible.host_key_checking = "false"
              ansible.limit = "all"
              ansible.galaxy_command = 'ansible-galaxy collection install community.mysql'
            end
          end
      end
  end
end
