# -*01- mode: ruby -*-
# vi: set ft=ruby :

servers = [
    {
        :name => "eerimez-k8s-master",
        :type => "master",
        :box => "ubuntu/xenial64",
        :box_version => "20180831.0.0",
        :eth1 => "192.168.205.30",
        :mem => "2048",
        :cpu => "2"
    },
	{
        :name => "eerimez-k8s-master2",
        :type => "master2",
        :box => "ubuntu/xenial64",
        :box_version => "20180831.0.0",
        :eth1 => "192.168.205.36",
        :mem => "2048",
        :cpu => "2"
    },
    {
        :name => "eerimez-k8s-worker-1",
        :type => "node",
        :box => "ubuntu/xenial64",
        :box_version => "20180831.0.0",
        :eth1 => "192.168.205.31",
        :mem => "2048",
        :cpu => "2"
    },
    {
        :name => "eerimez-k8s-worker-2",
        :type => "node",
        :box => "ubuntu/xenial64",
        :box_version => "20180831.0.0",
        :eth1 => "192.168.205.32",
        :mem => "2048",
        :cpu => "2"
    },
	 {
        :name => "eerimez-k8s-worker-3",
        :type => "node",
        :box => "ubuntu/xenial64",
        :box_version => "20180831.0.0",
        :eth1 => "192.168.205.33",
        :mem => "2048",
        :cpu => "2"
    },
	{
        :name => "eerimez-k8s-worker-4",
        :type => "node",
        :box => "ubuntu/xenial64",
        :box_version => "20180831.0.0",
        :eth1 => "192.168.205.34",
        :mem => "2048",
        :cpu => "2"
    },
	{
        :name => "eerimez-k8s-worker-5",
        :type => "node",
        :box => "ubuntu/xenial64",
        :box_version => "20180831.0.0",
        :eth1 => "192.168.205.35",
        :mem => "2048",
        :cpu => "2"
    }
]

# This script to install k8s using kubeadm will get executed after a box is provisioned
$configureBox = <<-SCRIPT

echo "install docker"
# install docker 
	sudo apt-get update -y
    sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common gnupg2
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
	#curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add --keyring /etc/apt/trusted.gpg.d/docker.gpg -
	
    sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
    sudo apt-get update -y
    sudo apt-get install -y docker-ce docker-ce-cli containerd.io
# Set up the Docker daemon
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

sudo mkdir -p /etc/systemd/system/docker.service.d
# Restart Docker
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo systemctl enable docker

# run docker commands as vagrant user (sudo not required)
    sudo usermod -aG docker vagrant

# install kubeadm
    sudo apt-get update && sudo apt-get install -y apt-transport-https gnupg2 curl
	curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
	sudo echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
    sudo apt-get update
    sudo apt-get install -y kubelet kubeadm kubectl 

#NOTES:	
#kubeadm: the command to bootstrap the cluster.
#kubelet: the component that runs on all of the machines in your cluster and does things like starting pods and containers.
#kubectl: the command line util to talk to your cluster.

# kubelet requires swap off
    swapoff -a

# keep swap off after reboot
    sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# restart the kubelet
   sudo systemctl restart kubelet
	
SCRIPT

$configureMaster = <<-SCRIPT
    echo "This is master"
# ip of this box
    IP_ADDR=`ifconfig enp0s8 | grep Mask | awk '{print $2}'| cut -f2 -d:`

# install k8s master 
	echo "install k8s master"
# https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/
    HOST_NAME=$(hostname -s)
    sudo kubeadm init --apiserver-advertise-address=$IP_ADDR --apiserver-cert-extra-sans=$IP_ADDR  --node-name $HOST_NAME --pod-network-cidr=172.16.0.0/16

echo "copying credentials to regular user - vagrant"
#copying credentials to regular user - vagrant
    sudo --user=vagrant mkdir -p /home/vagrant/.kube
    sudo cp -i /etc/kubernetes/admin.conf /home/vagrant/.kube/config
    sudo chown $(id -u vagrant):$(id -g vagrant) /home/vagrant/.kube/config

echo "install Calico pod network addon"
# install Calico pod network addon
    export KUBECONFIG=/etc/kubernetes/admin.conf
	kubectl create -f https://docs.projectcalico.org/manifests/tigera-operator.yaml
	
	curl https://docs.projectcalico.org/manifests/custom-resources.yaml > custom-resources.yaml
	#sudo sed -i "/^[^#]*cidr:[[:space:]]192.168.0.0/c\ cidr: 172.16.0.0/16/g" custom-resources.yaml
	sudo sed -i "s/"192.168.0.0/172.16.0.0/g custom-resources.yaml
    kubectl create -f custom-resources.yaml
	
echo "create token and save it"
# create token and save it
	kubeadm token create --print-join-command >> /etc/kubeadm_join_cmd.sh
    chmod +x /etc/kubeadm_join_cmd.sh
	
echo "required for setting up different ip internal"
# required for setting up different ip internal
    IP_ADDR=`ifconfig enp0s8 | grep Mask | awk '{print $2}'| cut -f2 -d:`
    sudo sed -i "/^[^#]*# the .N/c\Environment="KUBELET_EXTRA_ARGS=--node-ip=$IP_ADDR"" /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
    sudo systemctl daemon-reload
    sudo systemctl restart kubelet

	echo "required for setting up password less ssh between guest VMs"
    # required for setting up password less ssh between guest VMs
    sudo sed -i "/^[^#]*PasswordAuthentication[[:space:]]no/c\PasswordAuthentication yes" /etc/ssh/sshd_config
    sudo service sshd restart

SCRIPT

$configureMaster2 = <<-SCRIPT
    echo "This is master2"
    # ip of this box
    IP_ADDR=`ifconfig enp0s8 | grep Mask | awk '{print $2}'| cut -f2 -d:`

    # required for setting up password less ssh between guest VMs
    sudo sed -i "/^[^#]*PasswordAuthentication[[:space:]]no/c\PasswordAuthentication yes" /etc/ssh/sshd_config
    sudo service sshd restart

SCRIPT

$configureNode = <<-SCRIPT
    echo "This is worker"
    sudo apt-get install -y sshpass
    sshpass -p "vagrant" scp -o StrictHostKeyChecking=no vagrant@192.168.205.30:/etc/kubeadm_join_cmd.sh .
    sudo sh ./kubeadm_join_cmd.sh
   
    # required for setting up different ip internal
    IP_ADDR=`ifconfig enp0s8 | grep Mask | awk '{print $2}'| cut -f2 -d:`
    sudo sed -i "/^[^#]*# the .N/c\Environment="KUBELET_EXTRA_ARGS=--node-ip=$IP_ADDR"" /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
    sudo systemctl daemon-reload
    sudo systemctl restart kubelet


    # required for setting up password less ssh between guest VMs
    sudo sed -i "/^[^#]*PasswordAuthentication[[:space:]]no/c\PasswordAuthentication yes" /etc/ssh/sshd_config
    sudo systemctl restart sshd.service
  
SCRIPT

Vagrant.configure("2") do |config|

    servers.each do |opts|
        config.vm.define opts[:name] do |config|

            config.vm.box = opts[:box]
            config.vm.box_version = opts[:box_version]
            config.vm.hostname = opts[:name]
            config.vm.network :private_network, ip: opts[:eth1], name: "vboxnet12"

            config.vm.provider "virtualbox" do |v|

                v.name = opts[:name]
            	 v.customize ["modifyvm", :id, "--groups", "/eerimez_Kubernetes_Development"]
                v.customize ["modifyvm", :id, "--memory", opts[:mem]]
                v.customize ["modifyvm", :id, "--cpus", opts[:cpu]]

            end

            # we cannot use this because we can't install the docker version we want - https://github.com/hashicorp/vagrant/issues/4871
            #config.vm.provision "docker"

            config.vm.provision "shell", inline: $configureBox

#            if opts[:type] == "master"
#                config.vm.provision "shell", inline: $configureMaster
#				if opts[:type] == "master2"
#					config.vm.provision "shell", inline: $configureMaster2
#					else
#					config.vm.provision "shell", inline: $configureNode
#				end

            if opts[:type] == "master"
               config.vm.provision "shell", inline: $configureMaster
           else
               config.vm.provision "shell", inline: $configureNode
			end

        end

    end

end
