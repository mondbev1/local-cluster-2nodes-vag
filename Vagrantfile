# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.define "node1" do |node1|
	node1.vm.box = "ubuntu/bionic64"
	node1.vm.hostname = "node1"
  	node1.vm.network "private_network", ip: "172.32.10.10"
	node1.vm.provider "virtualbox" do |v|
  		v.memory = 4096
  		v.cpus = 2
	end
	node1.vm.provision :shell, privileged: false, inline: $provision_master_node
  end
  config.vm.define "node2" do |node2|
	node2.vm.box = "ubuntu/bionic64"
	node2.vm.hostname = "node2"
  	node2.vm.network "private_network", ip: "172.32.10.11"
	node2.vm.provision :shell, privileged: false, inline: <<-SHELL
sudo /vagrant/join.sh
SHELL
  end
  config.vm.provision :shell, privileged: false, inline: $install_common_tools
end


$install_common_tools = <<-SCRIPT
sudo swapoff -a
sudo sed -i '/swap/d' /etc/fstab
git clone https://github.com/otomato-gh/container.training.git
cd container.training
./prepare-vms/setup_kubeadm.sh
SCRIPT


$provision_master_node = <<-SHELL
OUTPUT_FILE=/vagrant/join.sh
rm -rf $OUTPUT_FILE
# Start cluster
sudo kubeadm init --node-name node1 --apiserver-advertise-address 172.32.10.10 --pod-network-cidr 10.1.0.0/16 | grep "kubeadm join" > ${OUTPUT_FILE}
echo -n " \
		--node-name node2
		" >> ${OUTPUT_FILE}
chmod +x $OUTPUT_FILE
# Configure kubectl
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
sudo su - $USER
sudo sysctl net.bridge.bridge-nf-call-iptables=1
export kubever=$(kubectl version | base64 | tr -d '\n')
kubectl apply -f https://cloud.weave.works/k8s/net?k8s-version=$kubever
SHELL
