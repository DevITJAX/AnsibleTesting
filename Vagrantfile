Vagrant.configure("2") do |config|
  servers=[
    {
      :hostname => "db01",
      :box => "bento/ubuntu-22.04",
      :ip => "192.168.169.130",
      :ssh_port => '2210'
    },
    {
      :hostname => "web01",
      :box => "bento/ubuntu-22.04",
      :ip => "192.168.169.131",
      :ssh_port => '2211'
    },
    {
      :hostname => "web02",
      :box => "bento/ubuntu-22.04",
      :ip => "192.168.169.132",
      :ssh_port => '2212'
    },
    {
      :hostname => "loadbalancer",
      :box => "bento/ubuntu-22.04",
      :ip => "192.168.169.134",
      :ssh_port => '2213'
    },
    {
      :hostname => "controlnode",
      :box => "bento/ubuntu-22.04",
      :ip => "192.168.169.135",
      :ssh_port => '2214'
    }
  ]

  config.vm.base_address = 600

  servers.each do |machine|
    config.vm.define machine[:hostname] do |node|
      node.vm.box = machine[:box]
      node.vm.hostname = machine[:hostname]

      # FIXED: use private_network instead of public_network
      node.vm.network "private_network", ip: machine[:ip]

      node.vm.network "forwarded_port", guest: 22, host: machine[:ssh_port], id: "ssh"

      node.vm.provider :virtualbox do |v|
        v.customize ["modifyvm", :id, "--memory", 2048]
        v.customize ["modifyvm", :id, "--name", machine[:hostname]]
      end

      if machine[:hostname] == "controlnode"
        node.vm.provision "shell", inline: <<-SHELL
          apt-get update
          apt-get install -y software-properties-common
          add-apt-repository --yes --update ppa:ansible/ansible
          apt-get install -y ansible
        SHELL
      end
    end
  end
end
