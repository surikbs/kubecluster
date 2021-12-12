IMAGE_NAME = "bento/ubuntu-20.10"
N = 2

Vagrant.configure("2") do |config|
    config.ssh.insert_key = false
    config.vm.provision "shell", inline: <<-SHELL
        apt-get update -y
        apt-get -y clean && sudo apt-get -y autoclean 
        apt install software-properties-common -y
        add-apt-repository --yes --update ppa:ansible/ansible -y
        apt install ansible -y
        apt-get -y clean && sudo apt-get -y autoclean 
        echo "192.168.50.10  k8s-master" >> /etc/hosts
        echo "192.168.50.11  worker-node-1" >> /etc/hosts
        echo "192.168.50.12  worker-node-2" >> /etc/hosts
    SHELL
    config.vm.provider "virtualbox" do |v|
        v.memory = 1024
        v.cpus = 2
    end
      
    config.vm.define "k8s-master" do |master|
        master.vm.box = IMAGE_NAME
        master.vm.network "private_network", ip: "192.168.50.10"
        master.vm.hostname = "k8s-master"
        master.vm.provision "ansible_local" do |ansible|
            ansible.playbook = "kubernetes-setup/master-playbook.yml"
            ansible.extra_vars = {
                node_ip: "192.168.50.10",
            }
        end
    end

    (1..N).each do |i|
        config.vm.define "worker-node-#{i}" do |node|
            node.vm.box = IMAGE_NAME
            node.vm.network "private_network", ip: "192.168.50.#{i + 10}"
            node.vm.hostname = "node-#{i}"
            node.vm.provision "ansible_local" do |ansible|
                ansible.playbook = "kubernetes-setup/node-playbook.yml"
                ansible.extra_vars = {
                    node_ip: "192.168.50.#{i + 10}",
                }
            end
        end
    end
end