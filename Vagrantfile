BOX_COUNT = 1
CPU_PER_BOX = 2
MEMORY_PER_BOX = 4096
IMAGE = "basebox_k3d_focal"

Vagrant.configure("2") do |config|
  (1..BOX_COUNT).each do |i|
    config.vm.define "k3d#{i}" do |k3ds|
      k3ds.vm.box = IMAGE
      k3ds.vm.provider :virtualbox do |v|
        v.linked_clone = true
        v.memory = MEMORY_PER_BOX
        v.cpus = CPU_PER_BOX
      end
      k3ds.vm.hostname = "k3d#{i}"

      #### Enable the folowing, if you want to shell into the box from another machine
      #### In thise case, add your public key to /home/vagrant/.ssh/authorized_keys by enabling the command abit further below
      #k3ds.vm.network "forwarded_port", guest: 22, host: 12222, protocol: "tcp"

      k3ds.vm.network  :private_network, ip: "10.0.0.#{i+10}"

      k3ds.vm.provision "shell", inline: <<-SHELL
        # Add current node in  /etc/hosts
        echo "127.0.1.1 $(hostname)" >> /etc/hosts

        k3d cluster create cluster1 --servers 3 --agents 1
        k3d node create cluster1-agent-1 --cluster cluster1 --role agent --replicas 3

        # Put the kube config in place, so kubectl can work without params
        mkdir -p /home/vagrant/.kube && \
		k3d kubeconfig get cluster1 > /home/vagrant/.kube/config && \
		chown -R vagrant:vagrant /home/vagrant/.kube

        # Bash Completion for kubectl - very handy
        kubectl completion bash >/etc/bash_completion.d/kubectl

        # Install OLM
        kubectl apply -f https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.17.0/crds.yaml
        kubectl apply -f https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.17.0/olm.yaml

        ### if you want to have connectivity over native ssh
	#echo "<YOUR_PUBLIC_KEY_HERE>" >> /home/vagrant/.ssh/authorized_keys
      SHELL
    end
  end
end
