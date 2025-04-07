# kubernetes-manual
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
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab<br>
cat <<EOF | tee /etc/modules-load.d/k8s.conf<br>
overlay<br>
br_netfilter<br>
EOF<br>
modprobe overlay<br>
modprobe br_netfilter<br>
### sysctl params required by setup, params persist across reboots
cat <<EOF | tee /etc/sysctl.d/k8s.conf<br>
net.bridge.bridge-nf-call-iptables  = 1<br>
net.bridge.bridge-nf-call-ip6tables = 1<br>
net.ipv4.ip_forward                 = 1<br>
EOF<br>
### Apply sysctl params without reboot
sysctl --system<br>
apt-get update<br>
apt-get install -y apt-transport-https ca-certificates curl gpg<br>
mkdir -p -m 755 /etc/apt/keyrings<br>
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg<br>
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | tee /etc/apt/sources.list.d/kubernetes.list<br>
apt-get update<br>
apt-get install -y kubelet kubeadm kubectl<br>
apt-mark hold kubelet kubeadm kubectl<br>
<br>
apt-get update<br>
apt-get install ca-certificates curl<br>
install -m 0755 -d /etc/apt/keyrings<br>
curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc<br>
chmod a+r /etc/apt/keyrings/docker.asc<br>
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  tee /etc/apt/sources.list.d/docker.list > /dev/null<br>
apt-get update<br>
apt-get install containerd.io docker-ce docker-ce-cli docker-buildx-plugin docker-compose-plugin -y<br>
apt update<br>
mkdir -p /etc/containerd<br>
containerd config default | tee /etc/containerd/config.toml<br>
sed -e 's/SystemdCgroup = false/SystemdCgroup = true/g' -i /etc/containerd/config.toml<br>
systemctl restart containerd<br>
systemctl enable containerd<br>
systemctl status containerd<br>
\`\`\`</code>

![image](https://github.com/user-attachments/assets/ef4b2538-20df-4e5d-8c7d-700c71158d19)
