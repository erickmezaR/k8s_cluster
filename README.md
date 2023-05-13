# k8s_cluster
A kubernetes in 1.27.1  cluster using calico and cri to play in your Vagrant environment

# How to use
1. setup your Vagrant directory
2. change your network on line 216 to match with your existing 
3. assign your Master node IP on lines 180, 11, 19,28, and 37
4. do a vagrant ip

If more nodes are required, you can edit the servers array in the Vagrantfile

```yaml
{
        :name => "k8s-worker-3",
        :type => "node",
        :box => "ubuntu/Jammy64",
        :box_version => "20180831.0.0",
        :eth1 => "192.168.205.33",
        :mem => "2048",
        :cpu => "2"
    }
```
    
you can choose the ram, cpu and ip for every VM

# Clean-up
vagrant destroy -f

# pause the cluster
vagrant halt
