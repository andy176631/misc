## Install Kubernetes and Configure Project Quota

This guide explains how to install Kubernetes on an Ubuntu 24.04 system and enable project quota support for local ephemeral storage.

### Steps Overview
1. Upgrade the kernel, as the VM’s Ubuntu kernel is incomplete.
2. Install CRI-O
3. Create a temporary virtual disk (dd), format it as ext4 with project quota, and mount it on /var/lib/kubelet.
4. Install kubeadm, kubelet, and kubectl
5. Run kubeadm init with the LocalStorageCapacityIsolationFSQuotaMonitoring feature gate enabled.
6. Install CNI


```bash
sudo apt update
sudo apt install linux-generic-hwe-24.04
reboot


export OS="xUbuntu_24.04"
export CRIO_VERSION="1.31"
curl -fsSL https://download.opensuse.org/repositories/isv:/kubernetes:/addons:/cri-o:/stable:/v${CRIO_VERSION}/deb/Release.key |
  sudo gpg --dearmor -o /usr/share/keyrings/cri-o-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/cri-o-archive-keyring.gpg] \
https://download.opensuse.org/repositories/isv:/kubernetes:/addons:/cri-o:/stable:/v${CRIO_VERSION}/deb/ /" \
  | sudo tee /etc/apt/sources.list.d/cri-o-${CRIO_VERSION}.list
sudo apt update
sudo apt install -y cri-o
sudo systemctl enable crio --now
sudo systemctl status crio


dd if=/dev/zero of=ext4-quota.img bs=1M count=100
mkfs.ext4 -O quota,project ext4-quota.img
mkdir -p /var/lib/kubelet
sudo mount -o loop,prjquota ./ext4-quota.img  /var/lib/kubelet


sysctl -w net.ipv4.ip_forward=1
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.34/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
sudo systemctl enable --now kubelet


echo 'KUBELET_EXTRA_ARGS="--feature-gates=LocalStorageCapacityIsolationFSQuotaMonitoring=true"' > /etc/default/kubelet
kubeadm init  --pod-network-cidr=10.244.0.0/16


curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm repo add cilium https://helm.cilium.io/
helm install cilium cilium/cilium --version 1.17.6   --namespace kube-system
```
