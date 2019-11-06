### try-cluster
使用kubeadm搭建K8s集群

xlh20385
Xu123

10.0.2.15 || 10.0.4.15 || 10.0.5.15

192.168.56.101 || 192.168.56.103 || 192.168.56.102
linhui.xu@uisee.com

搭建集群指南
下载centos.iso
配置两张网卡：NAT(上网) Host-Only(虚拟机，真机之间的通信)

yum install net-tools
yum -y install wget
使用阿里云的源
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo 
yum makecache

关闭防火墙
systemctl stop firewalld & systemctl disable firewalld

关闭Swap
执行swapoff -a可临时关闭，但系统重启后恢复
编辑/etc/fstab，注释掉包含swap的那一行即可，重启后可永久关闭
sed -i '/ swap / s/^/#/' /etc/fstab


安装docker(建议docker-ce-18.06.2.ce)
# Install Docker CE
## Set up the repository
### Install required packages.
yum install yum-utils device-mapper-persistent-data lvm2

### Add Docker repository.
yum-config-manager \
  --add-repo \
  https://download.docker.com/linux/centos/docker-ce.repo

## Install Docker CE.
yum update && yum install docker-ce-18.06.2.ce

## Create /etc/docker directory.
mkdir /etc/docker

# Setup daemon.
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
EOF

mkdir -p /etc/systemd/system/docker.service.d

# Restart Docker
systemctl daemon-reload
systemctl restart docker

安装 kubectl kubelet kubeadm
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

# Set SELinux in permissive mode (effectively disabling it)
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

systemctl enable --now kubelet

systemctl enable kubelet && systemctl start kubelet

cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sysctl --system

复制docker镜像到虚拟机(为docker 设置代理)

/etc/systemd/system/docker.service.d/http_proxy.conf

[Service]
Environment="HTTP_PROXY=http://192.168.56.1:12333/" "HTTPS_PROXY=http://192.168.56.1:12333/" "NO_PROXY=localhost,127.0.0.1,daocloud.io"

sudo systemctl daemon-reload
systemctl restart docker



终端设置代理
export ALL_PROXY=socks5://127.0.0.1:1080



k8s.images
k8s.gcr.io/kube-apiserver:v1.16.2
k8s.gcr.io/kube-controller-manager:v1.16.2
k8s.gcr.io/kube-scheduler:v1.16.2
k8s.gcr.io/kube-proxy:v1.16.2
k8s.gcr.io/pause:3.1
k8s.gcr.io/etcd:3.3.15-0
k8s.gcr.io/coredns:1.6.2





kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=10.0.2.15

kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=192.168.56.101



To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.56.101:6443 --token 0nqigm.p4s39qst1rfuqvqj \
    --discovery-token-ca-cert-hash sha256:cd12e96dfd507d6a9725c7dee06b33797433e0f49e969740a86c109465e7da95 


安装dashboard 需要安装在master节点上


使用过程中发现k9s无法shell进入其他容器当中，发现是node Internal-IP的设置问题(为NAT上的IP)，导致kubelet之间无法建立起访问
https://medium.com/@kanrangsan/how-to-specify-internal-ip-for-kubernetes-worker-node-24790b2884fd
https://stackoverflow.com/questions/54942488/how-to-change-the-internal-ip-of-kubernetes-worker-nodes

参考第二篇文章，进行修改/usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf
添加 为 KUBELET_CONFIG_ARGS 添加--node-ip=192.168.56.102


helm 3.0安装之后 要添加repo
kedacore	https://kedacore.azureedge.net/helm
stable  	http://mirror.azure.cn/kubernetes/charts

# apphub    	https://apphub.aliyuncs.com





