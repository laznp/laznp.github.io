---
permalink: /restart-deployment-on-secret-change-using-kyverno
---

# **Auto Restart Deployment on Secret Change**

## Install Kyverno
If you haven't install Kyverno, you check on this [link](/deploy-and-use-kyverno)

## Kyverno Policies for Auto Restart Deployment
### Create ClusterRoleBinding for kyverno so kyverno can restart deployment
Create file called `kyverno-deployment-crb.yaml` and paste code below
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kyverno:deployment
  labels:
    app: kyverno
subjects:
  - kind: ServiceAccount
    name: kyverno
    namespace: kyverno
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:controller:deployment-controller
```
NOTE! This is for testing purpose, never use `system:*:*` role for any service account for security reason.

Apply above manifest using `kubectl apply -f kyverno-deployment-crb.yaml`

### Apply kyverno policy for autorestart
Copy below code to `autorestart-deployment-policy.yaml`
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: restart-deployment-on-secret-change
  annotations:
    policies.kyverno.io/title: Restart Deployment On Secret Change
    policies.kyverno.io/category: other
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Deployment
    kyverno.io/kyverno-version: 1.7.0
    policies.kyverno.io/minversion: 1.7.0
    kyverno.io/kubernetes-version: "1.23"
    policies.kyverno.io/description: >-
      If Secrets are mounted in ways which do not naturally allow updates to
      be live refreshed it may be necessary to modify a Deployment. This policy
      watches a Secret and if it changes will write an annotation
      to one or more target Deployments thus triggering a new rollout and thereby
      refreshing the referred Secret. It may be necessary to grant additional privileges
      to the Kyverno ServiceAccount, via one of the existing ClusterRoleBindings or a new
      one, so it can modify Deployments.
spec:
  mutateExistingOnPolicyUpdate: true
  rules:
  - name: update-secret
    match:
      any:
      - resources:
          kinds:
          - Secret
          names:
          - nginx-secret
          namespaces:
          - default
    preconditions:
      all:
      - key: {% raw %}"{{request.operation}}"{% endraw %}
        operator: Equals
        value: UPDATE
    mutate:
      targets:
        - apiVersion: apps/v1
          kind: Deployment
          name: nginx
          namespace: default
      patchStrategicMerge:
        spec:
          template:
            metadata:
              annotations:
                deployment-version: {% raw %}"{{request.object.metadata.resourceVersion}}"{% endraw %}

```
Apply with `kubectl -f autorestart-deployment-policy.yaml`

For testing, we will create simple `nginx` deployment and secret.

Copy below command to your terminal
```sh
$ kubectl create secret generic nginx-secret --from-literal=domain=laznp.id --from-literal=owner=myself
```
Create `nginx-deployment.yaml` using below code and apply.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:latest
        name: nginx
        env:
          - name: DOMAIN
            valueFrom:
              secretKeyRef:
                name: nginx-secret
                key: domain
          - name: OWNER
            valueFrom:
              secretKeyRef:
                name: nginx-secret
                key: owner
```
Notice the pod name
```sh
$ kubectl get pod
NAME                     READY   STATUS    RESTARTS   AGE
nginx-7b4bd5656d-499zx   1/1     Running   0          18s
```
Follow below step to edit secret
```sh
$ echo "changedvalue" | base64
Y2hhbmdlZHZhbHVlCg== # copy this value

$ kubectl edit secret nginx-secret # paste copied value to one of the secret key
```
Lets check the deployment if it rollout new pod
```sh
$ kubectl get pod
NAME                     READY   STATUS        RESTARTS   AGE
nginx-7b4bd5656d-499zx   1/1     Terminating   0          4m6s
nginx-9f6485cfd-v4mcx    1/1     Running       0          4s
# notice the old pod is terminating and replaced with new one
```

That is all we need to do if we want to auto restart deployment on every secret changes. 
There are still many example policy Kyverno, you can check on their official document on [Kyverno](https://kyverno.io/policies/). Or you can create your own policy to harden you Kubernetes Cluster.

If you are not using kyverno, you can check another project called [Reloader](https://github.com/stakater/Reloader) with same purpose for restarting deployment on changed secret/configmap.