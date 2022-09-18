# vagrant-kubernetes-cluster
#Followed this link: 
https://prog.world/how-to-set-up-a-kubernetes-cluster-on-a-vagrant-vm/

How to set up a Kubernetes cluster on a Vagrant VM
In this Kubernetes tutorial, I went through a step-by-step guide to setting up a Kubernetes cluster on Vagrant. This is a multi-node Kubernetes setup using kubeadm…

Vagrant is a great utility for setting up virtual machines on your local workstation.


This tutorial is primarily about automated Kubernetes configuration using Vagrantfile and shell scripts.

Automatic configuration of a Kubernetes cluster on Vagrant
I wrote a basic Vagrantfile and scripts so that everyone can understand and make changes according to their requirements.

Configuration summary.

Team vagrant up will create three virtual machines and configure all basic kubernetes components and configuration using Kubeadm.

The kubeconfig file is added to all nodes in the cluster so you can run kubectl commands from any node.

The kubeconfig file and the kubernetes access token are added to the configs folder where you have your Vagrantfile. You can use the kubeconfig file to connect the cluster from your workstation.

You can shutdown virtual machines when not in use and restart them when needed. All cluster configurations remain intact without any problem. Nodes are automatically connected to the master during startup.

You can delete all virtual machines with one command vagrant destroy and recreate the installation using the command vagrant up Anytime.

Kubernetes, Kubeadm, Vagrant, Github Repository
Kubeadm, Vagrantfile and scripts are located in github repos…


Clone the repository.

git clone https://github.com/scriptcamp/vagrant-kubeadm-kubernetes
Configuring a Kubernetes cluster on Vagrant
Follow the steps below to deploy a Kubernetes cluster and verify all cluster configurations.

Step 1: To create a cluster, go to the cloned directory.

cd vagrant-kubeadm-kubernetes
Step 2: Run the vagrant command. She will expand three nodes. One master and two worker-nodes. Installing and configuring Kubernetes occurs through a bash script present in the scripts folder.

vagrant up
Note: If this is the first time you run it, Vagrant will first download the ubuntu image mentioned in the Vagrantfile. This is a one-time download.

Step 3: Login in master-node to check the cluster configuration.

vagrant ssh master
Step 4: List all nodes in the cluster to make sure worker-nodes are connected to master and ready.

kubectl top nodes
You should see what is shown below.


That’s all! You can start deploying and testing other applications.


To shutdown the Kubernetes VMs, run the command:

vagrant halt
When you need the cluster again, just do:

vagrant up
To remove virtual machines:

vagrant destroy -f
Explanation of Kubeadm, Vagrantfile and Scripts
The vagrant repository file tree.

├── Vagrantfile
├── configs
│   ├── config
│   ├── join.sh
│   └── token
└── scripts
├── common.sh
├── master.sh
└── node.sh
The configs folder and files are generated only after the first launch.

As I explained earlier, the config folder contains a config file, a token, and a join.sh file.


In the previous section, I already explained what config and token are. The join.sh file contains the command to join the worker-node with the token generated during the initialization of the master-node kubeadm.

Since all nodes share a folder with the Vagrantfile, worker-node can read the join.sh file and automatically join master during the first startup. This is a one-time task.


If you enter any node and access the folder /vagrant, you will see the Vagrantfile and scripts as they are shared across virtual machines.

Let’s take a look at Vagrantfile

Vagrant.configure("2") do |config|
config.vm.provision "shell", inline: <<-SHELL
apt-get update -y
echo "10.0.0.10  master-node" >> /etc/hosts
echo "10.0.0.11  worker-node01" >> /etc/hosts
echo "10.0.0.12  worker-node02" >> /etc/hosts
SHELL
config.vm.define "master" do |master|
  master.vm.box = "generic/ubuntu2004"
  master.vm.hostname = "master-node"
  master.vm.network "private_network", ip: "10.0.0.10"
  master.vm.provider "virtualbox" do |vb|
      vb.memory = 4048
      vb.cpus = 2
  end
  master.vm.provision "shell", path: "scripts/common.sh"
  master.vm.provision "shell", path: "scripts/master.sh"
end
(1..2).each do |i|
config.vm.define "node0#{i}" do |node|
node.vm.box = "generic/ubuntu2004"
node.vm.hostname = "worker-node0#{i}"
node.vm.network "private_network", ip: "10.0.0.1#{i}"
node.vm.provider "virtualbox" do |vb|
vb.memory = 2048
vb.cpus = 1
end
node.vm.provision "shell", path: "scripts/common.sh"
node.vm.provision "shell", path: "scripts/node.sh"
end
end
end
As you can see, I have added the following IP addresses for the nodes and it is added to the host file entry of all nodes with its hostname with a common shell block that runs on all virtual machines.

10.0.0.10 (master)

10.0.0.11 (node ​​01)

10.0.0.12 (node ​​02)

Also, the worker-node block is in a loop. Therefore, if you need more than two worker-nodes, or you only have one worker-node, you need to replace 2 with the desired number in the loop. If you are adding more nodes, make sure you add the IP to the node.

For example, for 3 worker-node you need to have a loop 1..4…

(1..4).each do |i|
master.sh, node.sh and common.sh
These three scripts are run as provisioners during Vagrant startup to configure the cluster.

common.sh: – List of commands that install docker, kubeadm, kubectl and kubelet on all nodes. It also disables swap.

master.sh: – contains commands for initializing master, installing calico plugin. Also copies the kube-config files, join.sh and token to the configs directory.

node.sh: – reads the command join.sh from the configs shared folder and attaches to master-node. Also copied the kubeconfig file to /home/vagrant/.kube location to execute kubectl commands.

Conclusion
To set up a kubernetes cluster on Vagrant, all you need to do is clone the repository and run the vagrant up command.

Also, you are a DevOps engineer and you work on a Kubernetes cluster, you can have a local development and testing stand.
