---
permalink: /security-analysis-kubesec
---

# **Security Analysis for Kubernetes Manifest with Kubesec**
Kubesec is useful for security analysis for your Kubernetes YAML Manifest.
You can combine it with your CI/CD Pipeline for check the manifest before its deployed to your cluster.

## Get Kubesec Binary
Download the latest Kubesec binary from their official [repo](https://github.com/controlplaneio/kubesec/releases) and extract.
```sh
$ cd $HOME
$ wget https://github.com/controlplaneio/kubesec/releases/download/v2.11.5/kubesec_darwin_amd64.tar.gz # I'm using MacOS so the binary is darwin based
$ tar xzvf kubesec_darwin_amd64.tar.gz
$ chmod +x kubesec
$ mv kubesec ~/.local/bin
```

## Create YAML Kubernetes Manifest
Copy below code snippet.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kubesec-demo
spec:
  containers:
  - name: kubesec-demo
    image: gcr.io/google-samples/node-hello:1.0
    securityContext:
      readOnlyRootFilesystem: true
EOF
```
Save above yaml as `kubesec-demo.yaml`

## Run Kubesec
For testing, run below command to check that our manifest will have some security issue
```sh
$ kubesec scan kubesec-test.yaml
```
You'll get this kind of output
```json
[
  {
    "object": "Pod/kubesec-demo.default",
    "valid": true,
    "fileName": "kubesec-test.yaml",
    "message": "Passed with a score of 1 points",
    "score": 1,
    "scoring": {
      "passed": [
        {
          "id": "ReadOnlyRootFilesystem",
          "selector": "containers[] .securityContext .readOnlyRootFilesystem == true",
          "reason": "An immutable root filesystem can prevent malicious binaries being added to PATH and increase attack cost",
          "points": 1
        }
      ],
      "advise": [
        {
          "id": "ApparmorAny",
          "selector": ".metadata .annotations .\"container.apparmor.security.beta.kubernetes.io/nginx\"",
          "reason": "Well defined AppArmor policies may provide greater protection from unknown threats. WARNING: NOT PRODUCTION READY",
          "points": 3
        },
        {
          "id": "ServiceAccountName",
          "selector": ".spec .serviceAccountName",
          "reason": "Service accounts restrict Kubernetes API access and should be configured with least privilege",
          "points": 3
        },
        {
          "id": "SeccompAny",
          "selector": ".metadata .annotations .\"container.seccomp.security.alpha.kubernetes.io/pod\"",
          "reason": "Seccomp profiles set minimum privilege and secure against unknown threats",
          "points": 1
        },
        {
          "id": "LimitsCPU",
          "selector": "containers[] .resources .limits .cpu",
          "reason": "Enforcing CPU limits prevents DOS via resource exhaustion",
          "points": 1
        },
        {
          "id": "LimitsMemory",
          "selector": "containers[] .resources .limits .memory",
          "reason": "Enforcing memory limits prevents DOS via resource exhaustion",
          "points": 1
        },
        {
          "id": "RequestsCPU",
          "selector": "containers[] .resources .requests .cpu",
          "reason": "Enforcing CPU requests aids a fair balancing of resources across the cluster",
          "points": 1
        },
        {
          "id": "RequestsMemory",
          "selector": "containers[] .resources .requests .memory",
          "reason": "Enforcing memory requests aids a fair balancing of resources across the cluster",
          "points": 1
        },
        {
          "id": "CapDropAny",
          "selector": "containers[] .securityContext .capabilities .drop",
          "reason": "Reducing kernel capabilities available to a container limits its attack surface",
          "points": 1
        },
        {
          "id": "CapDropAll",
          "selector": "containers[] .securityContext .capabilities .drop | index(\"ALL\")",
          "reason": "Drop all capabilities and add only those required to reduce syscall attack surface",
          "points": 1
        },
        {
          "id": "RunAsNonRoot",
          "selector": "containers[] .securityContext .runAsNonRoot == true",
          "reason": "Force the running image to run as a non-root user to ensure least privilege",
          "points": 1
        },
        {
          "id": "RunAsUser",
          "selector": "containers[] .securityContext .runAsUser -gt 10000",
          "reason": "Run as a high-UID user to avoid conflicts with the host's user table",
          "points": 1
        }
      ]
    }
  }
]
```
As you can see, Kubesec tell us which line that possibly against security policy and give us some advice to using Kubernetes Best Practices

For further information you can refer to [Kubesec](https://kubesec.io/) official documentation