Vagrant.configure("2") do |config|
  servers=[
    { hostname: "db01",         box: "bento/ubuntu-22.04", ip: "192.168.169.130", ssh_port: 2210 },
    { hostname: "web01",        box: "bento/ubuntu-22.04", ip: "192.168.169.131", ssh_port: 2211 },
    { hostname: "web02",        box: "bento/ubuntu-22.04", ip: "192.168.169.132", ssh_port: 2212 },
    { hostname: "loadbalancer", box: "bento/ubuntu-22.04", ip: "192.168.169.134", ssh_port: 2213 },
    { hostname: "controlnode",  box: "bento/ubuntu-22.04", ip: "192.168.169.135", ssh_port: 2214 }
  ]

  servers.each do |machine|
    config.vm.define machine[:hostname] do |node|
      node.vm.box      = machine[:box]
      node.vm.hostname = machine[:hostname]

      node.vm.network "private_network", ip: machine[:ip]
      node.vm.network "forwarded_port", guest: 22, host: machine[:ssh_port], id: "ssh"

      node.vm.provider :virtualbox do |v|
        v.customize ["modifyvm", :id, "--memory", 2048]
        v.customize ["modifyvm", :id, "--name", machine[:hostname]]
      end

      # Provision managed nodes to allow password authentication initially
      if machine[:hostname] != "controlnode"
        node.vm.provision "shell", inline: <<-SHELL
          # Enable password authentication for initial setup
          sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
          systemctl restart sshd
        SHELL
      end

      if machine[:hostname] == "controlnode"
        node.vm.provision "shell", inline: <<-SHELL
          apt-get update
          apt-get install -y ansible sshpass
          
          # Create ansible directory
          mkdir -p /etc/ansible
          
          # Generate SSH key for vagrant user (no passphrase)
          sudo -u vagrant ssh-keygen -t rsa -b 2048 -f /home/vagrant/.ssh/id_rsa -N ""
          
          # Set proper permissions
          chown -R vagrant:vagrant /home/vagrant/.ssh
          chmod 700 /home/vagrant/.ssh
          chmod 600 /home/vagrant/.ssh/id_rsa
          chmod 644 /home/vagrant/.ssh/id_rsa.pub
        SHELL
        
        # Copy the hosts file to the controlnode
        node.vm.provision "file", source: "./hosts", destination: "/tmp/hosts"
        node.vm.provision "shell", inline: "mv /tmp/hosts /etc/ansible/hosts"
      end
    end
  end
end
