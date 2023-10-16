# Setting Up KubeEdge Envrionment
## How to Install k8s and KubeEdge and set up the envirionment 
### My Envrionment setting 
<table class="center">
  <tr>
    <th></th>
    <th>Node1</th>
    <th>Node2</th>
    <th>Node3</th>
  </tr>
  <tr>
    <td>OS</td>
    <td colspan="3" style="text-align: center;">ubuntu 20.04.0 LTS</td> <!-- 이 셀은 가운데 정렬됩니다. -->
  </tr>
  <tr>
    <td>CORE(MEM)</td>
    <td>4Core(16GB)</td>
    <td>4Core(16GB)</td>
    <td>2Core(4GB)</td>
  </tr>
  <tr>
    <td>Kubernetes</td>
    <td colspan="2" style="text-align: center;">v.1.25.0</td> <!-- 이 셀도 가운데 정렬됩니다. -->
    <td></td> <!-- 이 셀은 비어 있지만, 마크업에서는 필요합니다. -->
  </tr>
  <tr>
    <td>KubeEdge</td>
    <td>v.1.12.4</td>
    <td></td> <!-- 이 셀은 비어 있습니다. -->
    <td>v.1.12.4</td>
  </tr>
  <tr>
    <td>ROLE</td>
    <td>k8s, ke Master</td>
    <td>k8s worker</td>
    <td>ke worker</td>
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
./k8s-cluster-bootstrap.sh -m -c 192.168.0.0/16 -i <cloud-IP> -ct containerd -v <kubernetes version>
```
#### 2. worker setup
```
sudo su
./k8s-cluster-bootstrap.sh -w -i <cloud-IP> -u <username> -p <password> -ct containerd -v <kubernetes version>
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
### 3.0 Installing keadm (Master | Worker )
```
wget https://github.com/kubeedge/kubeedge/releases/download/<version>/keadm-/<version>-linux-amd64.tar.gz
tar -zxvf keadm-/<version>-linux-amd64.tar.gz
cp keadm-/<version>-linux-amd64/keadm/keadm /usr/local/bin/keadm
```
### 3.1 Setting up Cloudcore
```
keadm init --advertise-address="<cloud-ip>" --profile version=<version> --kube-config=/root/.kube/config
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
Edit cloudCore deployment and add a Node Affinity, NodeSelector and Tolerance.
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
And Get token for Edgecore to join
```
keadm token > token.txt
```
### 3.2 Setting up EdgeCore
```
keadm join --cloudcore-ipport="<cloud-ip>":10000 --token=<token>
```



