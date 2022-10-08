Set up kubernetes cluster using kubeadm tool on Ubuntu 20.04LTS
These command will be executed on all the nodes

1. ---- create config file for containerd by adding the modules overlay and br_netfilter

cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
> overlay
> br_netfilter
> EOF


2.load the modules
sudo modprobe overlay
sudo modprobe br_netfilter

3. create system configuration for the kubernetes networking
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
>net.bridge.bridge-nf-call-iptables = 1
>net.ipv4.ip_forward = 1
>net.bridge.bridge-nf-call-ip6tables = 1
>EOF

4. Apply those settings
sudo sysctl --system

5. install containerd 
sudo apt-get update && sudo apt-get install -y containerd

6. create default configuration file for containerd
sudo mkdir -p /etc/containerd

7. Generate default config file and save it
sudo containerd config default | sudo tee /etc/containerd/config.toml

8. Restart containerd and verify it is running
sudo systemctl restart containerd
sudo systemctl status containerd

9. Disable swap
sudo swapoff -a

10. Make it permanent and add fstab
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

11. Install dependencies and all the packages
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
sudo apt-get update && sudo apt-get install -y apt-transport-https curl

12. Add Kubernetes to the repository list
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

13. Update the repo
sudo apt-get update

14. Install the kubernetes packages
sudo apt-get install -y kubelet kubeadm kubectl

===========
The command below will only be executed on Master plane

kubeadm init --pod-network-cidr=192.168.0.0/16

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
To choose a pod network of your choice, check https://kubernetes.io/docs/concepts/cluster-administration/addons/

Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.4.0.*:**** --token ************ \
        --discovery-token-ca-cert-hash sha256:*********************************************

If you lost the join command, use the below command to print it out
kubeadm token create --print-join-command
