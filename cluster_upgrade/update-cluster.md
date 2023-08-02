# Update Cluster

## Kind configuration
For this example we are using kind to simulate a kubernetes cluster. The file kind-config26, contains a kind configuration to create a cluster with 1 master and 2 workers of kubernetes version 26.

    # To create the cluster
    kind create cluster --config kind-config-26.yaml --name kind26

The cluster nodes are based on debian.

## Upgrade Node

In Kind, the kubeadm is not installed with apt. In order to install with apt follow the [instructions](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/):

    ## Install the tools
    apt-get update
    apt-get install -y apt-transport-https ca-certificates curl 
    apt-get install gpg

    # Create the folder to save keyrings for repository keys
    mkdir /etc/apt/keyrings

    # Download the repository key
    curl -fsSL https://dl.k8s.io/apt/doc/apt-key.gpg | gpg --dearmor -o /etc/apt/keyrings/kubernetes-archive-keyring.gpg

    # Add the repository to sources list
    echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | tee /etc/apt/sources.list.d/kubernetes.list

Install kubeadm:

    # kubeadm installation using apt
    apt-get update
    apt-get install -y kubeadm 
    apt-mark hold kubeadm
