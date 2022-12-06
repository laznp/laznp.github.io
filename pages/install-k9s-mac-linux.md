---
permalink: /install-k9s-mac-linux
---
# Installing K9s on Mac or Linux

K9s is an interactive Kubernetes terminal that makes it easy to monitor and manage your Kubernetes cluster. This tutorial will show you how to install K9s on Mac or Linux. 

## Prerequisites

* You must have the [Go Programming Language](https://golang.org/dl/) installed on your machine.
* You must have access to a Kubernetes cluster.

## Installation

1. Download the K9s binary for your platform:

| Platform | Command |
| -------- | ------- |
| Linux | `curl -L https://github.com/derailed/k9s/releases/latest/download/k9s_Linux_x86_64.tar.gz | tar zx` |
| Mac OS | `curl -L https://github.com/derailed/k9s/releases/latest/download/k9s_Darwin_x86_64.tar.gz | tar zx` |

2. Move the K9s binary to a directory in your `$PATH`. For example:

```sh
sudo mv k9s /usr/local/bin
```

3. Make sure the binary is executable:

```sh
sudo chmod +x /usr/local/bin/k9s
```

4. Connect to your Kubernetes cluster:

```sh
k9s cluster --kubeconfig <path_to_your_kubeconfig_file>
```

5. Start using K9s:

```sh
k9s
```

## Conclusion

You have now installed K9s on your Mac or Linux machine and connected to your Kubernetes cluster. You can start using K9s to manage and monitor your cluster.