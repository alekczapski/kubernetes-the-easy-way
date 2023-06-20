NUMBER_OF_NODES = 3

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"
  config.vm.box_version = "20230616.0.0"
  config.vm.box_check_update = false
  
  config.vm.provider "virtualbox" do |vb|
    vb.memory = 2048
    vb.cpus = 2
    vb.customize ["modifyvm", :id, "--nested-hw-virt", "on"]
    vb.customize ["modifyvm", :id, "--graphicscontroller", "vmsvga"]
    vb.check_guest_additions = false
  end
  
  config.vm.provision "shell", inline: <<-EOF
    snap install microk8s --classic --channel 1.27/stable
    microk8s status --wait-ready
    usermod -a -G microk8s vagrant
  EOF

  config.vm.define "node0", primary: true do |subconfig|
    subconfig.vm.hostname = "node-0"
    subconfig.vm.network :private_network, ip: "172.16.0.10"
    subconfig.vm.network "forwarded_port", guest: 80, host: 9080
    subconfig.vm.provision "shell", inline: <<-EOF
      echo "alias kubectl='microk8s kubectl'" >> /home/vagrant/.bashrc
      echo "alias helm3='microk8s helm3'" >> /home/vagrant/.bashrc
      runuser -l vagrant -c "mkdir -p ~/.kube && microk8s kubectl completion bash > ~/.kube/completion.bash.inc && echo -e '\nsource ~/.kube/completion.bash.inc' >> ~/.bashrc"
    EOF
    if NUMBER_OF_NODES > 1
      subconfig.vm.provision "shell", inline: <<-EOF
        microk8s add-node --token-ttl $((#{NUMBER_OF_NODES - 1} * 300)) | grep "172.16.0.10" > /vagrant/add-node.sh
        for I in {1..#{NUMBER_OF_NODES - 1}}; do echo "172.16.0.$((I + 10)) node-${I}" >> /etc/hosts; done
      EOF
    end
  end

  if NUMBER_OF_NODES > 1
    (1..NUMBER_OF_NODES - 1).each do |i|
      config.vm.define "node#{i}" do |subconfig|
        subconfig.vm.hostname = "node-#{i}"
        subconfig.vm.network :private_network, ip: "172.16.0.#{i + 10}"
        subconfig.vm.provision "shell", inline: <<-EOF
          sh /vagrant/add-node.sh
        EOF
      end
    end
  end

end
