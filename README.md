# rook-ceph
Rook Ceph Deployment and Testing

## Install Operator
```
helm repo add rook-release https://charts.rook.io/release
helm install --create-namespace --namespace rook-ceph rook-ceph rook-release/rook-ceph -f values.yaml
```

## Install Multus
Test deploy a pods with your mulutus definitions, sometimes there is some missing binary, or wrong hostpath.
```
git clone https://github.com/k8snetworkplumbingwg/multus-cni.git
cd multus-cni && cat ./deployments/multus-daemonset-thick.yml | kubectl apply -f -
```

## Install Whereabouts
```bash
git clone https://github.com/k8snetworkplumbingwg/whereabouts && cd whereabouts
kubectl apply     -f doc/crds/daemonset-install.yaml     -f doc/crds/whereabouts.cni.cncf.io_ippools.yaml     -f doc/crds/whereabouts.cni.cncf.io_overlappingrangeipreservations.yaml
```

## Deploy your ceph cluster

```bash
kubectl create -f multus/
kubectl create -f ceph-cluster/cluster-multus.yaml
kubectl create -f block-pools/ # Will create a RBD pools and Storage Class
```

## Test consume storage from CEPH
```bash
kubectl create -f test/
```

## Teardown the Clusters
```bash

kubectl delete -f test/ #Delete your workloads including the PVC
kubectl delete -f block-pools/ #Delete your StorageClass and pools 
kubectl delete -f ceph-cluster/cluster-multus.yaml #Delete your Ceph Cluster

###MAKE SURE YOU ARE CHANGE THE DISK Variables CORRECT###
# This will destroy your /var/lib/rook/* and Clean your DISK
# Once the pods is compleate, you need to delete the daemonset again
kubectl create -f tear-down/ 
```

## Reference
- https://static.sched.com/hosted_files/kccncna2021/0f/Storage_and_networking_Rook-Ceph_on_Multus_SebastienHan_RohanGupta_101321_v1.pdf

