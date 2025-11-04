vi /etc/apt/sources.list.d/kubernetes.list


---

sudo apt update
sudo apt-cache madison kubeadm

---

sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.33.0-1.1' && \
sudo apt-mark hold kubeadm

---

# replace x in 1.33.x-* with the latest patch version
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.33.0-1.1' kubectl='1.33.0-1.1' && \
sudo apt-mark hold kubelet kubectl


sudo systemctl daemon-reload
sudo systemctl restart kubelet

