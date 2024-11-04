# Setting up Kind on EC2
A kind is a tool for running local Kubernetes clusters using Docker container “nodes”. Kind was primarily designed for testing Kubernetes itself, but may be used for local development or CI.

![logo](https://github.com/user-attachments/assets/738051de-901d-465c-aec6-1196e73ad0d3)

# Pre-requisites:

VM with installed Docker https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-22-04

During the installation process we change the startup permissions, to make the changes take effect you need to run the command:
```
newgrp docker
```

# 1.  Update your package list:
```
sudo apt update
```

# 2. Install GoLang (required for kind):
```
sudo apt install -y golang
```
# 3. Install kubectl (Kubernetes command-line tool):
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```
# 4. Install KinD:
```
# For AMD64 / x86_64
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
# For ARM64
[ $(uname -m) = aarch64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-arm64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```
# 5. Create KinD config file config.yaml:
```
nano config.yaml
```
Paste the following content into the editor:
```
apiVersion: kind.x-k8s.io/v1alpha4
kind: Cluster
nodes:
- role: control-plane
- role: worker
    extraPortMappings:
      - containerPort: 30000
        hostPort: 30000
        listenAddress: "0.0.0.0" # Optional, defaults to "0.0.0.0"
        protocol: tcp # Optional, defaults to tcp
  ```
# 6. Create a Kubernetes cluster using KinD:
```
kind create cluster --config=config.yaml
```
![image](https://github.com/user-attachments/assets/bfd19457-5f56-4823-8275-126b37b328ed)

Wait a couple of minutes for the cluster to be fully created.

# 7. Add RoleBinding for kube-scheduler: This step avoids access issues by granting  kube-scheduler access to specific resources.
```
kubectl create rolebinding kube-scheduler-auth-reader \
    --namespace kube-system \
    --role=extension-apiserver-authentication-reader \
    --serviceaccount=kube-system:kube-scheduler
```
# 8. Verify Cluster Creation:
```
kubectl cluster-info --context kind-kind
```
![image](https://github.com/user-attachments/assets/1e720574-ceaa-4849-bae9-6c701d142755)

# 9. Test the Cluster:

kubectl get nodes
kubectl config view
kubectl get pods -A
kubectl get replicaset -A   
kubectl get deployments -A
kubectl get services -A
![image](https://github.com/user-attachments/assets/2a0a0ab3-4314-4339-b63b-24d4017759c9)

# References:
https://kind.sigs.k8s.io/docs/user/quick-start/#installation
https://aws.amazon.com/ec2/instance-types/t2/
