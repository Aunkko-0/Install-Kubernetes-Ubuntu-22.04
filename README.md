# Install-Kubernetes-Ubuntu-22.04 
### ส่วนใหญ่จะประกอบด้วย 
#### - 1 Master Node
#### - 2 Worket Node
---
1. Disable Swap (ทั้ง Worker และ Master)
```
swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```
2. Hostname (กรณีที่อยากเปลี่ยนชื่อ )
```
sudo hostnamectl set-hostname "control-plane"
exec bash
```
