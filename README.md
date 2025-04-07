# Kubernetes Cluster Manual - 1 Master Node & 1 Worker Node
This is a manual Kubernetes cluster setup for a non-production environment.
## Introduction
I am setting up a Kubernetes environment to gain a better understanding of how a cluster operates manually. For this purpose, I have created one master node and one worker node to simulate a sample environment.
## Azure virtual machine creation
![image](https://github.com/user-attachments/assets/9e152a2c-9ccc-4a63-8fc6-c41c2b555630)
## Connecting to the VM Using iTerm2 on macOS
![image](https://github.com/user-attachments/assets/011c9fba-5137-4f26-829f-125e9a9253e9)
Since I am using a terminal emulator for macOS (iTerm2), I split the pane vertically, enabled input broadcasting, and executed commands simultaneously on both the master and worker nodes.<br>
## Step-by-Step Implementation Instructions
<code>\`\`\` <br>
sudo su <br>
swapoff -a <br>
<br>
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab<br>
cat <<EOF | tee /etc/modules-load.d/k8s.conf<br>
overlay<br>
br_netfilter<br>
EOF<br>
<br>
modprobe overlay<br>
modprobe br_netfilter<br>\`\`\`</code>
### sysctl params required by setup, params persist across reboots
<code>\`\`\` cat <<EOF | tee /etc/sysctl.d/k8s.conf<br>
net.bridge.bridge-nf-call-iptables  = 1<br>
net.bridge.bridge-nf-call-ip6tables = 1<br>
net.ipv4.ip_forward                 = 1<br>
EOF<br>\`\`\`</code>
### Apply sysctl params without reboot
<code>\`\`\` sysctl --system<br>
apt-get update<br>
<br>
apt-get install -y apt-transport-https ca-certificates curl gpg<br>
<br>
mkdir -p -m 755 /etc/apt/keyrings<br>
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg<br>
<br>
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | tee /etc/apt/sources.list.d/kubernetes.list<br>
<br>
apt-get update<br>
apt-get install -y kubelet kubeadm kubectl<br>
apt-mark hold kubelet kubeadm kubectl<br>
<br>
apt-get update<br>
apt-get install ca-certificates curl<br>
install -m 0755 -d /etc/apt/keyrings<br>
<br>
curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc<br>
<br>
chmod a+r /etc/apt/keyrings/docker.asc<br>
<br>
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  tee /etc/apt/sources.list.d/docker.list > /dev/null<br>
<br>
apt-get update<br>
apt-get install containerd.io docker-ce docker-ce-cli docker-buildx-plugin docker-compose-plugin -y<br>
<br>
apt update<br>
<br>
mkdir -p /etc/containerd<br>
containerd config default | tee /etc/containerd/config.toml<br>
sed -e 's/SystemdCgroup = false/SystemdCgroup = true/g' -i /etc/containerd/config.toml<br>
systemctl restart containerd<br>
systemctl enable containerd<br>
systemctl status containerd<br>
\`\`\`</code>
![image](https://github.com/user-attachments/assets/ef4b2538-20df-4e5d-8c7d-700c71158d19)
### Run this only in Master node
<code>\`\`\`kubeadm init --apiserver-advertise-address $(hostname -i) --pod-network-cidr=192.168.0.0/16<br>
mkdir -p $HOME/.kube<br>
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config<br>
chown $(id -u):$(id -g) $HOME/.kube/config<br>
export KUBECONFIG=/etc/kubernetes/admin.conf<br>
<br>
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.1/manifests/calico.yaml<br>
\`\`\`</code><br>
![image](https://github.com/user-attachments/assets/83a394ec-f8c2-4c7b-b9b1-31254c4e6cbc)

### Run this only in Worker nodes
<code>\`\`\`kubeadm join 10.0.0.4:6443 --token h7xpru.8vw3d10qzzf7rffr \
	--discovery-token-ca-cert-hash sha256:4cb8bd267de0fdb2a2f9d9264e57f72dd4e60cfbd3c1ff81fa61f7413751677e\`\`\`</code><br>
 watch -n 1 kubectl get nodes

![image](https://github.com/user-attachments/assets/2e352a76-0d82-46c7-aac4-d3285548f90d)

## Challenges of Setting Up a Kubernetes Cluster Manually
 I found it difficult to connect the worker node to the master node while setting up the Kubernetes cluster. However, Mr. Vijay helped me by explaining the use of specific commands required for this task. He guided me on how to correctly execute the necessary commands to establish the connection, making the process much clearer.
 
## Conclusion
Setting up a Kubernetes cluster manually using kubeadm gives very useful hands-on experience. By learning how to set up and configure each part, developers can better understand how Kubernetes works. This helps in solving problems and improving the system later. Though it takes more time than using automatic tools, this method gives better knowledge and control over the Kubernetes setup.
