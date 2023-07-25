# Update Cluster

For this example we are using kind to simulate a kubernetes cluster. The file kind-config26, contains a kind configuration to create a cluster with 1 master and 2 workers of kubernetes version 26.

    # To create the cluster
    kind create cluster --config kind-config-26.yaml --name kind26

The cluster nodes are based on debian.
