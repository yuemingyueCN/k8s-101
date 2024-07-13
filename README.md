# k8s-101


# k8s install

## step1 containerd install

#### install containerd

~~~bash
tar Cxzvf /usr/local containerd-1.7.19-linux-amd64.tar.gz

# systemd
mkdir -p /usr/local/lib/systemd/system
mv containerd.service /usr/local/lib/systemd/system/containerd.service

systemctl daemon-reload
systemctl enable --now containerd
~~~

#### install runc

~~~ bash
$ install -m 755 runc.amd64 /usr/local/sbin/runc
~~~

#### install cni plugins

~~~ bash
mkdir -p /opt/cni/bin
tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.1.1.tgz
~~~

#### configure containerd

~~~ bash
mkdir /etc/containerd
containerd config default > /etc/containerd/config.toml

vi /etc/containerd/config.toml
# SystemdCgroup = true
# sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.9"
~~~

#### start containerd

~~~ bash
sudo systemctl daemon-reload
sudo systemctl enable --now containerd
sudo systemctl status containerd
~~~

## step2 Enable IPv4 packet forwarding

~~~ bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system
~~~

## step3 install k8s

~~~ bash
sudo apt-get update

sudo apt-get install -y apt-transport-https ca-certificates curl gpg

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
~~~

## step4 init

~~~ bash
kubeadm config images pull --image-repository registry.aliyuncs.com/google_containers
sudo kubeadm init  
 --image-repository registry.aliyuncs.com/google_containers \
 --apiserver-advertise-address=192.168.34.2 \
 --pod-network-cidr=10.10.0.0/16

kubeadm init  --image-repository registry.aliyuncs.com/google_containers  --apiserver-advertise-address=172.30.219.96  --pod-network-cidr=10.244.0.0/16 --ignore-preflight-errors=all
 
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
~~~

## step5 flannel

~~~ bash
wget https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
~~~

~~~ bash
  net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "vxlan"
      }
    }
###  ->
  net-conf.json: |
    {
      "Network": "10.10.0.0/16",
      "Backend": {
        "Type": "vxlan"
      }
    }
~~~

~~~ bash
kubectl apply -f kube-flannel.yml 
~~~

