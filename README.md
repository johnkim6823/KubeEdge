# Setting Up KubeEdge Envrionment
 How to Install k8s and KubeEdge and set up the envirionment 
### My Envrionment setting 
<table>
  <tr>
    <th align="center"></th>
    <th align="center">Node1</th>
    <th align="center">Node2</th>
    <th align="center">Node3</th>
  </tr>
  <tr>
    <td align="center">OS</td>
    <td colspan="3" align="center">ubuntu 20.04.0 LTS</td>
  </tr>
  <tr>
    <td align="center">CORE(MEM)</td>
    <td align="center">4Core(16GB)</td>
    <td align="center">4Core(16GB)</td>
    <td align="center">2Core(4GB)</td>
  </tr>
  <tr>
    <td align="center">Kubernetes</td>
    <td colspan="2" align="center">v.1.25.0</td>
    <td align="center"></td>
  </tr>
  <tr>
    <td align="center">KubeEdge</td>
    <td align="center">v.1.12.4</td>
    <td align="center"></td>
    <td align="center">v.1.12.4</td>
  </tr>
  <tr>
    <td align="center">ROLE</td>
    <td align="center">k8s, ke Master</td>
    <td align="center">k8s worker</td>
    <td align="center">ke worker</td>
  </tr>
</table>

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
./k8s-cluster-bootstrap.sh -m -c 192.168.0.0/16 -i <cloud-IP>  -v 1.25.0
```
#### 2. worker setup
```
sudo su
./k8s-cluster-bootstrap.sh -w -i <cloud-IP> -u <username> -p <password> -v 1.25.0
```
From Master get join command
```
kubeadm token create --print-join-command 
```
Put command in Worker node to join the cluster 
```
kubeadm join <cloud-ip>:6443 --token <token>
```
Restart the kubelet
```
systemctl restart kubelet
```

## Setting KubeEdge environment
### 3.0.1 Prerequisites
See Go is in local variable
```
go version
```
if not set it as local variable 
```
echo 'export PATH=$PATH:/usr/local/go/bin' >>${HOME_PATH}/.profile
echo 'export GOPATH=$HOME/go' >>${HOME_PATH}/.profile
source ${HOME_PATH}/.profile
mkdir -p $GOPATH
go version
```
or download
```
wget https://golang.org/dl/go1.20.5.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.20.5.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin' >>${HOME_PATH}/.profile
echo 'export GOPATH=$HOME/go' >>${HOME_PATH}/.profile
source ${HOME_PATH}/.profile
mkdir -p $GOPATH
go version
```
### 3.0 Installing keadm (Master | Worker )
```
wget https://github.com/kubeedge/kubeedge/releases/download/v1.12.4/keadm-v1.12.4-linux-amd64.tar.gz
tar -zxvf keadm-v1.12.4-linux-amd64.tar.gz
cp keadm-v1.12.4-linux-amd64/keadm/keadm /usr/local/bin/keadm
keadm version 
```

### 3.1 Setting up Cloudcore
```
keadm init --kubeedge-version=1.12.4 --kube-config=/root/.kube/config --advertise-address=<cloudcore ip>
```
If it successed
```
Kubernetes version verification passed, KubeEdge installation will start...
CLOUDCORE started
=========CHART DETAILS=======
NAME: cloudcore
LAST DEPLOYED: Wed Oct 26 11:10:04 2022
NAMESPACE: kubeedge
STATUS: deployed
REVISION: 1
```
Ensure that cloudcore start successfully.
```
kubectl get all -n kubeedge
```
If it successed, then pod Status must be Running not CrushLoopBack and service/Cloudcore must be exists
```
NAME                             READY   STATUS    RESTARTS   AGE
pod/cloudcore-56b8454784-ngmm8   1/1     Running   0          46s

NAME                TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)                                             AGE
service/cloudcore   ClusterIP   10.96.96.56   <none>        10000/TCP,10001/TCP,10002/TCP,10003/TCP,10004/TCP   46s

NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cloudcore   1/1     1            1           46s

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/cloudcore-56b8454784   1         1         1       46s
```
Next, we need to add a NodeLabel to the master node so that we can utilize the NodeLabel to schedule the CloudCore to the Master node.
```
kubectl label node <master <node name> node-role.kubernetes.io/nodeType=cloudCore
```
#Edit cloudCore deployment and add a Node Affinity, NodeSelector and Tolerance.
kubectl edit deployment -n kubeedge cloudcore
```
~~~~
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: node-role.kubernetes.io/nodeType
          operator: In
          values:
          - cloudCore
tolerations:
- key: "node-role.kubernetes.io/control-plane"
  operator: "Exists"
  effect: "NoSchedule"
- key: "node-role.kubernetes.io/master"
  operator: "Exists"
  effect: "NoSchedule"
nodeSelector:
  node-role.kubernetes.io/nodeType: cloudCore
```
Check the kubeedge version

And Get token for Edgecore to join
```
keadm token > token.txt
```
### 3.2 Setting up EdgeCore
```
keadm join --cloudcore-ipport=<cloudcore_ip>:10000 --token=<token>
```



