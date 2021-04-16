# -*- mode: ruby -*- 
# vi: set ft=ruby :

LAB_NAME = "CHANGEME"
K8S_BOX = "geerlingguy/ubuntu1804"
MASTER_NODE_COUNT = 1
WORKER_NODE_COUNT = 1
# Last octect
IP_SEED=0
MASTER_NET4_START = 20
NODE_NET4_START = 30

LB_BOX = "giovannibaratta/alpine313python"
LB_VIP_NET4 = 10

## DEBUG VARIABILE. DO NOT MODIFY !!
RUN_LB_ANSIBLE = true

# Seed random
srand(IP_SEED)

# Second and third octect
net2_random = rand(0..255)
net3_random = rand(0..255)

Vagrant.configure("2") do |config|

    loadbalancer_vip = "10.#{net2_random}.#{net3_random}.#{LB_VIP_NET4}"
    loadbalancer_cidr = "10.#{net2_random}.#{net3_random}.#{LB_VIP_NET4}/24"

    # Generate node ip array
    loadbalancer_ip_array = Array.new(2) { |index| "10.#{net2_random}.#{net3_random}.#{LB_VIP_NET4 + index + 1}" }
    master_ip_array = Array.new(3) { |index| "10.#{net2_random}.#{net3_random}.#{MASTER_NET4_START + index}" }
    workers_ip_array = Array.new(3) { |index| "10.#{net2_random}.#{net3_random}.#{NODE_NET4_START + index}" }

    master_api_ip = MASTER_NODE_COUNT == 1 ? master_ip_array[0] : loadbalancer_vip

    if MASTER_NODE_COUNT > 1
    # Deploy di 2 vm per l'HAProxy

        (1..2).each do |index|
            config.vm.define "loadbalancer#{index}" do |node|
                node.vm.box = LB_BOX
                node.vm.hostname = "loadbalancer#{index}"
                node.vm.network "private_network", ip: loadbalancer_ip_array[index-1]
                node.vm.provider "virtualbox" do |loadbalancer|
                    loadbalancer.name = "#{LAB_NAME}-loadbalancer#{index}"
                    loadbalancer.cpus = 1
                    loadbalancer.memory = 256
                end
                
                if index == 2 and RUN_LB_ANSIBLE
                   # Trigger ansible configuration
                    node.vm.provision "ansible" do |ansible|
                        ansible.playbook = "ansible/loadbalancer_main.yaml"
                        ansible.limit = ["loadbalancer1", "loadbalancer2"]
                        ansible.groups = {
                        "master" => ["loadbalancer1"],
                        "backup" => ["loadbalancer2"]
                        }
                        ansible.extra_vars = {
                            loadbalancer_vip: loadbalancer_vip,
                            loadbalancer_cidr: loadbalancer_cidr,
                            master_node_ip: master_ip_array
                        }
                        ansible.verbose = "true"
                    end
                end
            end
        end
    end # Load balancer end

    # Deploy master nodes
    (1..MASTER_NODE_COUNT).each do |index|
        config.vm.define "master#{index}" do |node|
            node.vm.box = K8S_BOX
            node.vm.hostname = "master#{index}"
            node.vm.network "private_network", ip: master_ip_array[index-1]
            node.vm.provider "virtualbox" do |master|
                master.name = "#{LAB_NAME}-k8s-master#{index}"
                master.cpus = 2
                master.memory = 2048
            end  
        end 
    end # Masters end

    # Deploy worker nodes
    (1..WORKER_NODE_COUNT).each do |index|
        config.vm.define "worker#{index}" do |node|
            node.vm.box = K8S_BOX    
            node.vm.hostname = "worker#{index}"
            node.vm.network "private_network", ip: workers_ip_array[index-1]
            node.vm.provider "virtualbox" do |worker|
                worker.name = "#{LAB_NAME}-k8sWorker#{index}"
                worker.cpus = 1
                worker.memory = 1024
            end
        
            if index == WORKER_NODE_COUNT
                # Trigger ansible configuration
                node.vm.provision "ansible" do |ansible|
                    ansible.playbook = "ansible/kubernetes_main.yaml"
                    ansible.limit = ["masters", "workers"]
                    ansible.groups = {
                        "masters_manager" => ["master1"],
                        "masters" => ["master[1:#{MASTER_NODE_COUNT}]"],
                        "workers" => ["worker[1:#{WORKER_NODE_COUNT}]"]
                    }
                    ansible.extra_vars = {
                        master_api_ip: master_api_ip
                    }
                    ansible.verbose = "true" 
                end
            end
        end
    end # Workers end
end
