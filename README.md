# Kubernetes-Demo1
# Kubernetes-Installation-Using KOPs on EC2 t3micro which is paid service

**Setup an EC2 instance using AWS**

Dependencies required for this setup are:
  1. Python3
  2. AWS CLI
  3. kubectl

**Install Correct Repositories for Ubuntu**

```
sudo mkdir -p /etc/apt/keyrings
```

```
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo tee /etc/apt/keyrings/kubernetes-apt-keyring.asc > /dev/null
```

```
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.asc] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

**Update package list and Install Kubernetes Components**

```
sudo apt-get update
```

```
sudo apt install -y kubelet kubeadm kubectl
```

```
sudo apt update && sudo apt install -y python3-pip
```


**Install AWS CLI which is unavailable on Ubuntu latest version**

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
```

```
unzip awscliv2.zip
```

```
sudo ./aws/install
```

```
aws  --version
```

```
source awscli-venv/bin/activate   ##AWS CLI environment should activated
```

If there is any error regarding unzip command unavailable use the unzip command and try again
apt install unzip 


**Verify Kubernetes Installation**

```
kubectl version --client && kubeadm version
```

**Configure AWS on the Instance**

```
aws configure
```

Enter the reuired Access Keys for AWS initialisations which are available in AWS profile and security
 1. AWS Access Key ID
 2. AWS Secret Access Key
 3. Default Region (eg: ap-south-1)
 4. Output format (default: json)

**Install KOPS**

```
curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
```

```
chmod +x kops-linux-amd64
```

```
sudo mv kops-linux-amd64 /usr/local/bin/kops
```

##verify kops installation

```
kops version
```

```
export PATH="$PATH:/home/ubuntu/.local/bin/"
```

```
echo 'export PATH="$PATH:/home/ubuntu/.local/bin/"' >> ~/.bashrc
```

```
source ~/.bashrc
```

```
echo $PATH
```


**Kubernetes Cluster Installation**
These steps help in installing cluster within  our instance region

**Creating S3 buckets for storing KOPS objects**

```
aws s3api create-bucket --bucket kops-kowshik-storage --region ap-south-1 --create-bucket-configuration LocationConstraint=ap-south-1
```

**Create Cluster**

```
kops create cluster --name=demok8scluster.k8s.local --state=s3://kops-kowshik-storage --zones=ap-south-1a --node-count=1 --node-size=t3.medium --control-plane-size=t3.medium --control-plane-volume-size=8  --node-volume-size=8
```

```
kops edit cluster demok8scluster.k8s.local
```

```
export KOPS_STATE_STORE=s3://kops-kowshik-storage
```

```
kops update cluster demok8scluster.k8s.local --yes --state=s3://kops-kowshik-storage
```

##This will update the storage as local storage which takes 10-15 minutes and verify the cluster using the below command

```
kops validate cluster demok8scluster.k8s.local
```

```
kubectl get nodes
```
will list the running nodes in the cluster

```
vim pod.yaml
```
Opens the Yaml file for the pod created

##Basic YAML script for the pod

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
````

This creates a private pod which is accessible with in the cluster. To expose it to the external ip we have to attach either load balancer or node port service to this pod using a Yaml file

```
vim service.yaml
```

###Basic YAML script for the service

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx  # Matches pods with label app=nginx
  ports:
    - protocol: TCP
      port: 80  # Service Port
      targetPort: 80  # Pod Port
  type: NodePort  # Exposes service externally
```

```
kubectl apply -f pod.yaml && kubectl apply -f service.yaml
```
Apply the pod changes and runs the pod and services

```
kubectl get pods
```
will lists the running pods in the node

```
kubectl get pods -o wide
```
returns the complete information about pod

```
kubectl get svc
```
will lists the running services in the node


```
curl https://<nodeport public ip> or <loadbalancer>:port
```
Connects to the node within the cluster

To connect using external browser

```
http://<nodeport public ip>
```

```
kubectl get deploy
```
will lists the no.of deployed files

```
vim deployment.yaml
```
Opens the yaml file for the deployment creation

##Basic Yaml script for deployment

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```


```
kubectl apply -f deployment.yaml
```
Apply the changes to deployment file and will run it

```
kubectl get rs
```
will list all created replicas by controller.





