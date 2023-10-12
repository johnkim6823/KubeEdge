# Setting Up KubeEdge Envrionment
How to Install k8s and KubeEdge and set up the envirionment
|            | Node1         | Node2         | Node3       | 
|------------|---------------|---------------|-------------|
| Kubernetes | v.1.25.0      | v.1.25.0      |             |
| KubeEdge   | v.1.12        |               | v.1.12      |
| ROLE       | k8s, ke Master| k8s worker    | ke worker   |


## Setting kubernetes envrionment
### 0. Prerequisites
#### 0.1 Get bootstrap.sh from ###
#### 0.2 Move shell script to home directroy
```
cd k8s-cluster-bootstrap
mv k8s-cluster-bootstrap.sh ../
```
#### 0.3 change mode
```
chmod +x k8s-cluster-bootstrap.sh 
```
#### 1. master setup
```
sudo su
./k8s-cluster-bootstrap.sh -m -c 192.168.0.0/16 -i <cloud-IP> -ct containerd -v <kubernetes version>
```
#### 2. worker setup
```
sudo su
./k8s-cluster-bootstrap.sh -w -i <cloud-IP> -u <username> -p <password> -ct containerd -v <kubernetes version>
```

## Setting KubeEdge environment
### 3. Installing keadm (Master | Worker )
```
wget https://github.com/kubeedge/kubeedge/releases/download/<version>/keadm-/<version>-linux-amd64.tar.gz
tar -zxvf keadm-/<version>-linux-amd64.tar.gz
cp keadm-/<version>-linux-amd64/keadm/keadm /usr/local/bin/keadm
```
### 3.1 Setting up Cloudcore
```
keadm init --advertise-address="<cloud-IP" --profile version=<version> --kube-config=/root/.kube/config
```
```
kubectl get all -n kubeedge
```
### 3.2 Setting up EdgeCore




