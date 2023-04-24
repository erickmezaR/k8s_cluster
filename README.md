# k8s_cluster
a kubernetes cluster using calico

# How to use
setup your Vagrant directory and start the  Kubernetes cluster, this will start one master and five nodes:

vagrant up

You can also start invidual machines by vagrant up <VM name>, vagrant up
  
  

If more nodes are required, you can edit the servers array in the Vagrantfile

{
        :name => "eerimez-k8s-worker-3",
        :type => "node",
        :box => "ubuntu/xenial64",
        :box_version => "20180831.0.0",
        :eth1 => "192.168.205.33",
        :mem => "2048",
        :cpu => "2"
    }
    
you can choose the ram, cpu and ip for every VM

# Clean-up
vagrant destroy -f

# pause the cluster
vagrant halt
