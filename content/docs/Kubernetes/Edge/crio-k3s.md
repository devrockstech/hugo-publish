---
title: Setting K3S With CRI-O Runtime
weight: 20
---
# Setting up Master Node

You need to run this script to setup Master node

```bash
#!/bin/bash
MASTER_IP=192.168.1.233
NODE1_IP=192.168.1.33
#NODE2_IP=
# Set hostname
echo $MASTER_IP master >> /etc/hosts
echo $NODE1_IP worker1 >> /etc/hosts
#echo $NODE2_IP worker2 >> /etc/hosts
# Set version
export OS=xUbuntu_20.04
export VERSION=1.19
export INSTALL_K3S_VERSION="v1.29.12%2Bk3s2"
# install cri-o and dependencies
echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /" > /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
echo "deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$VERSION/$OS/ /" > /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.list

curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$VERSION/$OS/Release.key | apt-key add -
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | apt-key add -

apt-get update
apt-get install -y cri-o cri-o-runc git golang
#remove swap space
swapoff -a
sed -i.bak -r 's/(.+ swap .+)/#\1/' /etc/fstab
#disable ip_v6
modprobe overlay
modprobe br_netfilter

cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sysctl --system
#Config CRIO
sed -i 's/system.slice/pod/g' /etc/crio/crio.conf
sed -i 's/systemd/cgroupfs/g' /etc/crio/crio.conf
cat <<EOF >>  /etc/crio/crio.conf
registries = [
  "quay.io",
  "docker.io"
]
EOF
curl https://raw.githubusercontent.com/cri-o/cri-o/master/contrib/cni/11-crio-ipv4-bridge.conf > /etc/cni/net.d/100-crio-bridge.conf
#Install CNI Plugin
git clone https://github.com/containernetworking/plugins
cd plugins
git checkout v0.8.7
./build_linux.sh 
sudo mkdir -p /opt/cni/bin
sudo cp bin/* /opt/cni/bin/


#Install k3s

systemctl restart crio
export K3S_KUBECONFIG_MODE="644"
export INSTALL_K3S_EXEC=" --container-runtime-endpoint /var/run/crio/crio.sock --no-deploy servicelb --no-deploy traefik"

curl -sfL https://get.k3s.io | sh -
#Set token in node script
TOKEN=$(cat /var/lib/rancher/k3s/server/node-token)
sed -i "s/MASTER_TOKEN/$TOKEN/g" k3s-crio-node.sh
#Install Helm Tools
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
helm repo add stable https://charts.helm.sh/stable
helm repo update

# Allow ports
#sudo ufw allow 6443/tcp
#sudo ufw allow 443/tcp
```