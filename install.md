
# k8s install

## 主机配置

~~~ bash
hostnamectl set-hostname k8s-master
~~~

#### ipv4 桥接

tar -zxvf cri-containerd-cni-1.7.18-linux-amd64.tar.gz -C /



 mkdir /etc/containerd -p

```
containerd config default > /etc/containerd/config.toml
```

```
vi /etc/containerd/config.toml
# SystemdCgroup = true
# sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.9"
```

systemctl enable containerd

systemctl start containerd

ctr images ls

runc --version、



cat <<EOF > /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

阿里源安装

root@k8s-master:~# modprobe overlay
root@k8s-master:~# modprobe br_netfilter

apt-get update && apt-get install -y apt-transport-https
curl -fsSL https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.28/deb/Release.key |
    gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.28/deb/ /" |
    tee /etc/apt/sources.list.d/kubernetes.list
apt-get update
apt-get install -y kubelet kubeadm kubectl





apt-mark hold kubelet kubeadm kubectl



修改这个

mkdir /etc/sysconfig

vim /etc/sysconfig/kubelet

KUBELET_EXTRA_ARGS="--cgroup-driver=systemd"



systemctl enable kubelet

```
kubeadm config images pull --image-repository registry.aliyuncs.com/google_containers
```



crictl image ls

sudo kubeadm init  
 --kubernetes-version=v1.30.2
 --image-repository registry.aliyuncs.com/google_containers \
 --apiserver-advertise-address=172.30.219.96 \
 --pod-network-cidr=10.224.0.0/16

--ignore-preflight-errors

~~~bash
sudo kubeadm init
 --kubernetes-version=v1.30.2
 --image-repository registry.aliyuncs.com/google_containers \
 --apiserver-advertise-address=172.30.219.96 \
 --pod-network-cidr=10.224.0.0/16 \
 --ignore-preflight-errors=all
 
 kubeadm init --kubernetes-version=v1.30.2 --pod-network-cidr=10.224.0.0/16 --apiserver-advertise-address 172.30.219.96 --image-repository registry.aliyuncs.com/google_containers --ignore-preflight-errors=all
~~~



下载callico

wget --no-check-certificate https://raw.githubusercontent.com/projectcalico/calico/v3.27.3/manifests/calico.yaml

vim calico.yaml 

            - name: CALICO_IPV4POOL_CIDR
              value: "10.244.0.0/16"
            - name: IP_AUTODETECTION_METHOD
              value: "interface=eth0"

ctr -n k8s.io image pull docker.io/calico/cni:v3.27.3

ctr -n k8s.io image pull docker.io/calico/node:v3.27.3

ctr -n k8s.io image pull docker.io/calico/kube-controllers:v3.27.3

https://docker.aityp.com/image/docker.io/calico/cni:v3.27.3



ctr images pull swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/calico/node:v3.27.3
ctr images tag  swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/calico/node:v3.27.3  docker.io/calico/node:v3.27.3

calico的镜像下载之后

kubectl apply -f calico.yaml

基本都启动了
