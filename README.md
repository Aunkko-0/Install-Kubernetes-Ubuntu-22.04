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
#### เช่น master-node IP 192.168.x.xxx
####  worker1-node IP 192.168.x.xxx
####  worker2-node IP 192.168.x.

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
