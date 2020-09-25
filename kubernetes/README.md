### Set up volumes ###

**All the commands are prepared for being executed from the kubernetes folder of the project**

For setting up volumes, first, the location of the volume in local file systems needs to be set
by editing **local-storage-files-nodex.yaml** in **ledger** folder:

```
  hostPath:
    path: <<ADD HERE YOUR PATH FOR FILES>>
```

Then, create and claim the file storage for each node:

```
cd
kubectl apply -f ./ledger/local-storage-files-node0.yaml
kubectl apply -f ./ledger/local-storage-claim-node0.yaml
kubectl apply -f ./ledger/local-storage-files-node1.yaml
kubectl apply -f ./ledger/local-storage-claim-node1.yaml
kubectl apply -f ./ledger/local-storage-files-node2.yaml
kubectl apply -f ./ledger/local-storage-claim-node2.yaml
```

You can check the volumes added to kubernetes with the next command:

```
kubectl get pv
```

And the claims:
```
kubectl get pvc
```

### Create the configmaps for nodes ###

Next, the configmap shuld be added to kubernetes.

First, the envoy config is added:
```
kubectl create configmap envoy-besu-config --from-file=./envoy/envoy.yaml
```

Then, add the besu config files:
```
kubectl create configmap besu-config --from-file=../common
```

Last, add the key files for the nodes:
```
kubectl create secret generic node0Key --from-file=./node0/private
kubectl create secret generic node1Key --from-file=./node1/private
kubectl create secret generic node2key --from-file=./node2/private
```

### Create the services ###

Now, the services can be created for the kubernetes cluster:
```
kubectl apply -f ./node0/node0-service.yaml
kubectl apply -f ./node1/node1-service.yaml
kubectl apply -f ./node2/node2-service.yaml
```

To set up correctly the node init, the ip from each node needs to be get from
the kubernetes cluster, run the command:
```
kubectl get services
```
It returns somethign like this:
```
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)              AGE
node0        ClusterIP   10.101.168.28    <none>        8545/TCP,30000/UDP   18h
node1        ClusterIP   10.104.151.20    <none>        8545/TCP,30000/UDP   17h
node2        ClusterIP   10.106.154.117   <none>        8545/TCP,30000/UDP   17h
```
In the deployment file (local-nodeX-deployment.yaml) change the parameter:
```
- --bootnodes=enode://9683f07e51ed54f30a58332db6cdabc0f2413127961d2ca11bfa637a091a6f738c5f0f977d3cfbc0b603f7cf78b3b05f1c6d58d3f9b7d37d1b3198590e1d5ade@<<CLUSTER-IP node0>>:30000
```
This needs to be done in node 1 and node 2.

### Create the deployments ###

Last, the deploy files are applied:
```
kubectl apply -f ./node0/local-node0-deployment.yaml
kubectl apply -f ./node1/local-node0-deployment.yaml
kubectl apply -f ./node2/local-node0-deployment.yaml
```

### Check pods ###
Pods can be checked now:
```
kubectl get pods
```