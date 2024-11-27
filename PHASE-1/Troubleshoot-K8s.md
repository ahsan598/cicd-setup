#  I. Troubleshoot K8-Cluster: Kubernetes has already been initialized or a previous initialization process has left residual files, ports, or directories in use


**Example:**
```sh
[preflight] Running pre-flight checks error execution phase preflight: [preflight] Some fatal errors occurred:
        [ERROR Port-6443]: Port 6443 is in use
        [ERROR FileAvailable--etc-kubernetes-manifests-kube-apiserver.yaml]: /etc/kubernetes/manifests/kube-apiserver.yaml already exists
```

## Run on Master node [run on Worker node as well if required]

### 1. Stop Kubernetes Services
```sh
sudo systemctl stop kubelet
sudo systemctl stop docker
```


### 2. Clean Up Previous Kubernetes Setup
```sh
sudo kubeadm reset -f
sudo rm -rf /etc/kubernetes
sudo rm -rf /var/lib/etcd
```

### 3. Install net-tools & Check and Free Ports
```sh
sudo apt install net-tools -y
sudo netstat -tuln | grep 6443
sudo netstat -tuln | grep 10250
```

### 4. Restart Kubectl & re-initialized Kubernetes
```sh
sudo systemctl restart kubelet
sudo kubeadm init --ignore-preflight-errors=all
```

### 5. Verify Cluster Status
```sh
kubectl get nodes
```



#  II. Troubleshoot K8-Cluster: An issue with the TLS certificate used by the Kubernetes API server. This usually happens due to mismatched or invalid certificates on the client or server side.


**Example:**
```sh
Unable to connect to the server: tls: failed to verify certificate: x509: certificate signed by unknown authority (possibly because of "crypto/rsa: verification error" while trying to verify candidate authority certificate "kubernetes")
```

## Run on Master node [run on Worker node as well if required]

### 1. Check Your kubeconfig File
```sh
kubectl config view
```

### 2. Re-create kubeconfig
```sh
sudo kubeadm init phase kubeconfig admin
sudo cp /etc/kubernetes/admin.conf ~/.kube/config
sudo chown $(id -u):$(id -g) ~/.kube/config
```

### 3. Check & Re-generate Certificates
```sh
ls /etc/kubernetes/pki/ca.crt
openssl x509 -in /etc/kubernetes/pki/ca.crt -noout -text

sudo kubeadm init phase certs all
sudo kubeadm init phase kubeconfig admin
sudo cp /etc/kubernetes/admin.conf ~/.kube/config
```

### 4. Restart kubelet
```sh
sudo systemctl restart kubelet
```

### 5. Verify Cluster Status
```sh
kubectl get nodes
```



#  III. Troubleshoot K8-Cluster: Error indicates that kubectl is unable to connect to the Kubernetes API server running on `Your-IP:6443`. This can happen due to various reasons such as the API server not running, network issues, or incorrect configuration.


**Example:**
```sh
The connection to the server 'Your-IP:6443' was refused - did you specify the right host or port?
```

## Run on Master node [run on Worker node as well if required]

### 1. Check if the Kubernetes API Server is Running
```sh
sudo systemctl status kubelet
sudo systemctl start kubelet
sudo systemctl enable kubelet
```

### 2. Verify kubeconfig & Re-generate kubeconfig
```sh
cat ~/.kube/config
sudo kubeadm init phase kubeconfig admin
mkdir -p $HOME/.kube
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### 3. Test & Verify kubectl
```sh
kubectl get nodes
kubectl cluster-info
```

### 4. Reinitialize the Cluster (If Necessary)
```sh
sudo kubeadm reset
sudo kubeadm init


mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### 5. Verify cluster
```sh
kubectl get nodes
kubectl cluster-info
```