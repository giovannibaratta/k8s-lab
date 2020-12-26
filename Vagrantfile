# -*- mode: ruby -*- 
# vi: set ft=ruby :

LAB_NAME = "CHANGEME"
BOX = "geerlingguy/ubuntu1804"
NODE_COUNT = 2

NET2_RANDOM = rand(0..255)
NET3_RANDOM = rand(0..255)

Vagrant.configure("2") do |config|

    # Creo il master
    config.vm.define "master" do |node|
        node.vm.box = BOX
        node.vm.hostname = "master"
        node.vm.network "private_network", ip: "10.#{NET2_RANDOM}.#{NET3_RANDOM}.10"
        
        node.vm.provider "virtualbox" do |master|
            master.name = "K8sMaster-#{LAB_NAME}"
            master.cpus = 2
            master.memory = 4096
        end
    end
    
    # Creo i worker
    (1..NODE_COUNT).each do |index|
        config.vm.define "worker#{index}" do |node|
            node.vm.network "private_network", ip: "10.#{NET2_RANDOM}.#{NET3_RANDOM}.#{ 10 + index }"
            node.vm.box = BOX
            node.vm.hostname = "worker#{index}"
            
            node.vm.provider "virtualbox" do |worker|
                worker.name = "K8sWorker#{index}-#{LAB_NAME}"
                worker.cpus = 2
                worker.memory = 2048
            end
        
	    if index == NODE_COUNT
                # Lancio il provisioning di ansible per tutte le VM
                node.vm.provision "ansible" do |ansible|
                   ansible.playbook = "setupK8s.yaml"
                   ansible.limit = "all"
                   ansible.groups = {
                     "master" => ["master"],
                     "worker" => ["worker[1:#{NODE_COUNT}]"]
                   }
                   #ansible.extra_vars = {
		   #	master_ip: "10.#{NET2_RANDOM}.#{NET3_RANDOM}.10"
                   #}
                end
            end
         end
     end  

end
