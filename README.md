# Rook-Ceph Deployment Guide

Welcome to the Rook-Ceph Deployment and Testing Guide. This document provides step-by-step instructions to deploy Rook-Ceph in your Kubernetes cluster, alongside additional networking components like Multus and Whereabouts for advanced network configurations.

## Getting Started

### Install the Rook-Ceph Operator

1. **Add the Rook Helm Repository**: First, add the Rook repository to your Helm repositories.
    ```bash
    helm repo add rook-release https://charts.rook.io/release
    ```
   
2. **Install the Rook-Ceph Operator**: Use Helm to install the Rook-Ceph operator in the `rook-ceph` namespace. Make sure to specify your configuration through a `values.yaml` file.
    ```bash
    helm install --create-namespace --namespace rook-ceph rook-ceph rook-release/rook-ceph -f values.yaml
    ```

### Install Multus

Multus CNI allows you to attach multiple network interfaces to pods in Kubernetes.

1. **Clone the Multus Repository**: Get the Multus CNI source code.
    ```bash
    git clone https://github.com/k8snetworkplumbingwg/multus-cni.git
    ```
   
2. **Deploy Multus**: Apply the Multus DaemonSet to your cluster.
    ```bash
    cd multus-cni
    kubectl apply -f ./deployments/multus-daemonset-thick.yml
    ```

### Install Whereabouts

Whereabouts provides IP address management (IPAM) for pods across multiple network interfaces.

1. **Clone the Whereabouts Repository**: Download the source code for Whereabouts.
    ```bash
    git clone https://github.com/k8snetworkplumbingwg/whereabouts && cd whereabouts
    ```
   
2. **Deploy Whereabouts**: Apply the necessary Kubernetes resources for Whereabouts.
    ```bash
    kubectl apply -f doc/crds/daemonset-install.yaml -f doc/crds/whereabouts.cni.cncf.io_ippools.yaml -f doc/crds/whereabouts.cni.cncf.io_overlappingrangeipreservations.yaml
    ```

## Deploying Your Ceph Cluster

1. **Prepare Network Configurations**: If you have custom network configurations, apply them first.
    ```bash
    kubectl create -f multus/
    ```

2. **Deploy the Ceph Cluster**: Use the provided manifest to create your Ceph cluster.
    ```bash
    kubectl create -f ceph-cluster/cluster-multus.yaml
    ```

3. **Create RBD Pools and Storage Class**: This step sets up the storage pools and classes for your Ceph cluster.
    ```bash
    kubectl create -f block-pools/
    ```

## Testing Ceph Storage

To verify that your Ceph storage is correctly set up and accessible:

```bash
kubectl create -f test/
```

## Cleaning Up

To tear down your deployment and clean up resources:

1. **Remove Test Resources**: Delete any workloads and their persistent volume claims (PVCs).
    ```bash
    kubectl delete -f test/
    ```
   
2. **Delete RBD Pools and Storage Classes**: Clean up the storage setup.
    ```bash
    kubectl delete -f block-pools/
    ```
   
3. **Delete the Ceph Cluster**: Remove the Ceph cluster deployment.
    ```bash
    kubectl delete -f ceph-cluster/cluster-multus.yaml
    ```

4. **Clean Disks and Remove Rook Data**: Make sure to adjust the `DISK` variables correctly before running this. It will clean your specified disks and remove Rook data. Once complete, remember to delete the daemonset.
    ```bash
    kubectl create -f tear-down/
    ```

## Reference
- [Storage and Networking: Rook on Multus - Sébastien Han & Rohan Gupta, Red Hat](https://static.sched.com/hosted_files/kccncna2021/0f/Storage_and_networking_Rook-Ceph_on_Multus_SebastienHan_RohanGupta_101321_v1.pdf).
