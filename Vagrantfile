# -*- mode: ruby -*-
# vi: set ft=ruby :

# this is the reference page
# https://www.linuxtechi.com/install-kubernetes-on-ubuntu-22-04/
servers = [
    {
        :name => "k8s-master",
        :type => "master",
        :box => "ubuntu/jammy64",
        :eth1 => "192.168.56.30",
        :mem => "2048",
        :cpu => "4"
    },
    {
        :name => "k8s-worker-1",
        :type => "node",
        :box => "ubuntu/jammy64",
        :eth1 => "192.168.56.31",
        :mem => "3072",
        :cpu => "4"
    },
    {
        :name => "k8s-worker-2",
        :type => "node",
        :box => "ubuntu/jammy64",
        #:box_version => "20180831.0.0",
        :eth1 => "192.168.56.32",
        :mem => "3072",
        :cpu => "4"
    },
	 {
        :name => "k8s-worker-3",
        :type => "node",
        :box => "ubuntu/jammy64",
        #:box_version => "20180831.0.0",
        :eth1 => "192.168.56.33",
        :mem => "3072",
        :cpu => "4"
    },
	{
        :name => "k8s-worker-4",
        :type => "node",
        :box => "ubuntu/jammy64",
        #:box_version => "20180831.0.0",
        :eth1 => "192.168.56.34",
        :mem => "3072",
        :cpu => "4"
    },
#    {
#        :name => "k8s-worker-5",
#        :type => "node",
#        :box => "ubuntu/jammy64",
#        #:box_version => "20180831.0.0",
#        :eth1 => "192.168.56.35",
#        :mem => "3072",
#        :cpu => "4"
#    }
]

# This script to install k8s using kubeadm will get executed after a box is provisioned
$configureBox = <<-SCRIPT

#disable swap and add kernel settings
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

#Load the following kernel modules on all the nodes
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
 
sudo modprobe overlay
sudo modprobe br_netfilter

#Set the following Kernel parameters for Kubernetes, run beneath
cat <<EOF | sudo tee /etc/sysctl.d/kubernetes.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

#Reload the above changes, run
sudo sysctl --system

#install containerd runtime
sudo apt-get install -y gnupg2 > /dev/null 2>&1 
sudo apt-get install -y ca-certificates > /dev/null 2>&1
sudo apt-get install -y lsb-release > /dev/null 2>&1
sudo apt-get install -y net-tools > /dev/null 2>&1
sudo apt-get install -y apt-transport-https > /dev/null 2>&1
sudo apt-get install -y software-properties-common > /dev/null 2>&1
sudo apt-get install -y curl > /dev/null 2>&1
echo "after isntall net-tools"

sudo install -m 0755 -d /etc/apt/keyrings
sudo usermod -aG docker vagrant

#Enable docker repository
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

#Now, run following apt command to install containerd
sudo apt update
sudo apt install -y containerd.io  > /dev/null 2>&1

#Configure containerd so that it starts using systemd as cgroup.
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml

#Restart and enable containerd service
sudo systemctl restart containerd
sudo systemctl enable containerd

#add apt repository for kubernetes
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/kubernetes-xenial.gpg
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"

#install kubernetes componenets kubectl kubeadm kubelet
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

SCRIPT

$configureMaster = <<-SCRIPT
echo "This is master"
# get the IP from master node
IP_ADDR=`ifconfig enp0s8 | grep -i Mask | awk '{print $2}'| cut -f2 -d:`
 
# get the hostname from master node
HOST_NAME=$(hostname -s)

#Initialize Kubernetes cluster with Kubeadm command IP 172.16.0.0 is independent of 192.168.56.x and does not affect
sudo kubeadm init --apiserver-advertise-address=$IP_ADDR --apiserver-cert-extra-sans=$IP_ADDR  --node-name $HOST_NAME --pod-network-cidr=172.16.0.0/16
echo "after init"

#to start interacting with cluster, run following commands from the master node
#copying credentials to regular user - vagrant
sudo --user=vagrant mkdir -p /home/vagrant/.kube
sudo cp -i /etc/kubernetes/admin.conf /home/vagrant/.kube/config
sudo chown $(id -u vagrant):$(id -g vagrant) /home/vagrant/.kube/config

#grant permission to root user to do kubectl
export KUBECONFIG=/etc/kubernetes/admin.conf

echo "after copy credentials"

#install Calico
kubectl apply -f "https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml"

sleep 20

#generate token to make workers node join the cluster
kubeadm token create --print-join-command >> /etc/kubeadm_join_cmd.sh
chmod +x /etc/kubeadm_join_cmd.sh
     
# required for setting up different ip internal
IP_ADDR=`ifconfig enp0s8 | grep -i Mask | awk '{print $2}'| cut -f2 -d:`
sudo sed -i "/^[^#]*# the .N/c\Environment="KUBELET_EXTRA_ARGS=--node-ip=$IP_ADDR"" /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

#restart services 
sudo systemctl daemon-reload
sudo systemctl restart kubelet
     
# required for setting up password less ssh between guest VMs
sudo sed -i "/^[^#]*PasswordAuthentication[[:space:]]no/c\PasswordAuthentication yes" /etc/ssh/sshd_config
sudo service sshd restart

SCRIPT

$configureNode = <<-SCRIPT
echo "This is worker"

# install sshpass
sudo apt-get install -y sshpass > /dev/null 2>&1

#copy the token file from master and execute to join the cluster
sshpass -p "vagrant" scp -o StrictHostKeyChecking=no vagrant@192.168.56.30:/etc/kubeadm_join_cmd.sh .
sh ./kubeadm_join_cmd.sh
   
# required for setting up different ip internal
IP_ADDR=`ifconfig enp0s8 | grep Mask | awk '{print $2}'| cut -f2 -d:`
sudo sed -i "/^[^#]*# the .N/c\Environment="KUBELET_EXTRA_ARGS=--node-ip=$IP_ADDR"" /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

#restart services
sudo systemctl daemon-reload
sudo systemctl restart kubelet


# required for setting up password less ssh between guest VMs
sudo sed -i "/^[^#]*PasswordAuthentication[[:space:]]no/c\PasswordAuthentication yes" /etc/ssh/sshd_config
sudo systemctl restart sshd.service

# to test your cluster
# kubectl get nodes
# kubectl create deployment nginx-app --image=nginx --replicas=2
# kubectl get deployment nginx-app
# kubectl expose deployment nginx-app --type=NodePort --port=80
# kubectl get svc nginx-app
# kubectl describe svc nginx-app #from here you will get the port in NodePort
# Use following command to access nginx based application,
# curl http://<woker-node-ip-addres>:31246
  
SCRIPT

Vagrant.configure("2") do |config|

    servers.each do |opts|
        config.vm.define opts[:name] do |config|

            config.vm.box = opts[:box]
            config.vm.box_version = opts[:box_version]
            config.vm.hostname = opts[:name]
            config.vm.network :private_network, ip: opts[:eth1], name: "vboxnet0" #remember this vboxnet0 is defined in your virtualbox

            config.vm.provider "virtualbox" do |v|

                v.name = opts[:name]
            	 v.customize ["modifyvm", :id, "--groups", "/eerimez_Kubernetes_Development"]
                v.customize ["modifyvm", :id, "--memory", opts[:mem]]
                v.customize ["modifyvm", :id, "--cpus", opts[:cpu]]

            end

            # we cannot use this because we can't install the docker version we want - https://github.com/hashicorp/vagrant/issues/4871
            #config.vm.provision "docker"

            config.vm.provision "shell", inline: $configureBox

            if opts[:type] == "master"
                config.vm.provision "shell", inline: $configureMaster
            else
                config.vm.provision "shell", inline: $configureNode
            end

        end

    end

end
