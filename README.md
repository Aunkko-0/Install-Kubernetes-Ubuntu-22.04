# Install-Kubernetes-Ubuntu-22.04 
### คำเตือน ถ้าจะติดตั้งกรุณาตั้งค่าพื้นที่ใน Server
---
###
ต้องมี 2 CPUs
มี RAM อย่างน้อย 2GB
ต้องมี Disk ว่างอย่างน้อยๆ 2 GB (ไม่รวมกับพื้นที่ ที่ application และ database ต้องใช้งาน)
###
---
### โดยปกติแล้วการใช้ Kubernetes จะใช้แบบ Kubernetes Cluster จะประกอบด้วย
#### - 1 Master Node
#### - 2 Worket Node
#### เช่น master-node IP 192.168.1.156
####  worker1-node IP 192.168.1.168
####  worker2-node IP 192.168.1.169

---
1. Disable Swap (ทั้ง Worker และ Master)
```
swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```
1.1. Hostname (ทั้ง Worker และ Master)
```
sudo hostnamectl set-hostname "control-plane" หรือ "master-node"
exec bash
```
1.2. เพิ่ม Kernel Parameters
```
sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter
```
1.3. ตั้งค่า Kernel Parameters
```
sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system
```
2. ติดตั้ง Docker (ทำทั้ง master & worker nodes)
```
# Add Docker's official GPG key
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```
รันคำสั่ง 
```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
ตั้งค่า Configure containerd
```
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
```
3. ติดตั้ง kubeadm, kubectl, kubelet (ทำทั้ง master & worker nodes)
เพิ่ม signing key
```
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
จากนั้นติดตั้ง kubelet kubeadm kubect
```
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubect
```
Lock version prevent auto-update
```
sudo apt-mark hold kubelet kubeadm kubectl
```
แทนค่า <NODE_IP> ตาม ip ของ node นั้น ๆ เช่น 192.168.1.156 ถ้าทำบน master-node และ 192.168.1.168 ถ้าทำบน worker1-node และ 192.168.1.168 ถ้าทำบน worker2-node
```
echo "KUBELET_EXTRA_ARGS=--node-ip=<NODE_IP>" | sudo tee /etc/default/kubelet
sudo systemctl daemon-reload
sudo systemctl restart kubelet
sudo systemctl restart containerd
```
4. Setup master node
Setup Master Node (ทำแค่ที่ master node)
แทนค่า <MASTER_NODE_IP> เป็น ip ในกรณีนี้คือ 192.168.1.156
```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=<MASTER_NODE_IP>
```
เมื่อสำเร็จจะได้ข้อความแนว ๆ นี้ พร้อมกับให้เรา copy command ของการ join cluster ไว้เตรียมใช้กับ worker nodes
<img width="842" height="48" alt="image" src="https://github.com/user-attachments/assets/ff5dd6fe-c9d8-4099-bc43-9e0fa5ceeb2c" />
ให้ copy คำสั่งตามนี้ที่แสดงในหน้าจอ
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
ติดตั้ง Network plugin ให้กับ Cluster ซึ่งประเภทของ Network plugin มีหลายประเภท ไปหาข้อมูลเอานะจ๊ะ 
```
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml
```
```
kubectl get node
```

