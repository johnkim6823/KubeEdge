# Setting Up KubeEdge Envrionment
How to Install k8s and KubeEdge and set up the envirionment

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

