---
permalink: /kubernetes-using-k3s
---

# **Deploy Kubernetes Cluster with k3s**

Kubernetes is an open-source container-orchestration system for automating application deployment, scaling, and management. k3s is a lightweight Kubernetes distribution designed for resource-constrained environments such as edge computing, CI/CD, and IoT.

This tutorial will guide you through the process of deploying a Kubernetes cluster with k3s.

## Prerequisites

* Linux machine with Docker installed

## Install k3s

1. Download the k3s binary:

```sh
curl -sfL https://get.k3s.io | sh -
```

2. The k3s server will start running immediately after installation.

3. To check if the server is running:

```sh
sudo systemctl status k3s
```

## Join the cluster

1. To join a node to the cluster, you will need to obtain the token from the server:

```sh
sudo cat /var/lib/rancher/k3s/server/node-token
```

2. Run the following command on the node you want to join, replacing <TOKEN> with the token obtained in the previous step:

```sh
curl -sfL https://get.k3s.io | K3S_TOKEN=<TOKEN> sh -
```

3. Once the node has joined the cluster, you can check the status of the nodes:

```sh
kubectl get nodes
```

## Deploy an application

1. To deploy an application, you will need to create a Deployment object. This can be done with the following command:

```sh
kubectl apply -f <PATH_TO_YAML_FILE>
```

2. Once the Deployment is created, you can check the status of the pods:

```sh
kubectl get pods
```

3. When the pods are in a running state, the application is deployed and ready to use!

## Conclusion

You have successfully deployed a Kubernetes cluster with k3s and deployed an application to it. With k3s, you can easily manage your containerized applications in a lightweight and resource-efficient way.