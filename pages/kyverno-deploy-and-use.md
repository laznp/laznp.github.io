---
permalink: /deploy-and-use-kyverno
---

# **Install and Use Kyverno Policies**

## Install Kyverno
### Add Kyverno Helm Repo
```sh
$ helm repo add kyverno https://kyverno.github.io/kyverno/
$ helm repo update
```
### Install Kyverno Standalone
```sh
$ helm install kyverno kyverno/kyverno -n kyverno --create-namespace
```

### Install Kyverno HA Mode
```sh
$ helm install kyverno kyverno/kyverno -n kyverno --create-namespace --set replicaCount=3
```

## Kyverno Policies Example
### Restrict Node Port
Create file called `restrict-nodeport.yaml` and paste code below
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: restrict-nodeport
  namespace: poc
  annotations:
    policies.kyverno.io/title: Disallow NodePort
    policies.kyverno.io/category: Best Practices
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Service
    policies.kyverno.io/description: >-
      A Kubernetes Service of type NodePort uses a host port to receive traffic from
      any source. A NetworkPolicy cannot be used to control traffic to host ports.
      Although NodePort Services can be useful, their use must be limited to Services
      with additional upstream security checks. This policy validates that any new Services
      do not use the `NodePort` type.
spec:
  validationFailureAction: enforce
  background: true
  rules:
  - name: validate-nodeport
    match:
      resources:
        kinds:
        - Service
    validate:
      message: "Services of type NodePort are not allowed."
      pattern:
        spec:
          type: "!NodePort"
```
Apply above manifest using `kubectl apply -f restrict-nodeport.yaml`

For testing, create file called `nodeport.yaml` and copy.
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort
  selector:
    app: MyApp
  ports:
    - port: 80
      targetPort: 80
```
Apply the manifest using `kubectl create -f nodeport.yaml` and 
You'll catch error like this
```
Error from server: error when creating "nodeport.yaml": admission webhook "validate.kyverno.svc-fail" denied the request:

resource Service/default/my-service was blocked due to the following policies

restrict-nodeport:
  validate-nodeport: 'validation error: Services of type NodePort are not allowed.
    Rule validate-nodeport failed at path /spec/type/'
```
And thats because Kyverno denied your requesto to create nodeport

### Sync Secret to Created Namespace
Below example is to sync secret to the newly created namespace. Base secret that we use is docker registry credential that we will create on `default` namespace.
```sh
$ kubectl create secret docker-registry regcred \
  --docker-email=tiger@acme.example \
  --docker-username=tiger \
  --docker-password=pass1234 \
``` 
Confirm that secret is created.
```sh
$ kubectl get secret -n default
NAME      TYPE                             DATA   AGE
regcred   kubernetes.io/dockerconfigjson   1      12s
```
Copy below code to `sync-secret.yaml`
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: sync-secrets
  annotations:
    policies.kyverno.io/title: Sync Secrets
    policies.kyverno.io/category: Sample
    policies.kyverno.io/subject: Secret
    policies.kyverno.io/description: >-
      Secrets like registry credentials often need to exist in multiple
      Namespaces so Pods there have access. Manually duplicating those Secrets
      is time consuming and error prone. This policy will copy a
      Secret called `regcred` which exists in the `default` Namespace to
      new Namespaces when they are created. It will also push updates to
      the copied Secrets should the source Secret be changed.
spec:
  rules:
  - name: sync-image-pull-secret
    match:
      resources:
        kinds:
        - Namespace
    generate:
      kind: Secret
      name: regcred
      namespace: {% raw %} "{{request.object.metadata.name}}" {% endraw %}
      synchronize: true
      clone:
        namespace: default
        name: regcred
```
Apply with `kubectl -f sync-secret.yaml`

For testing, create namespace and check the automatically synced secret
```sh
$ kubectl create ns testing-sync-secret
namespace/testing-sync-secret created

$ kubectl get secret -n testing-sync-secret
NAME      TYPE                             DATA   AGE
regcred   kubernetes.io/dockerconfigjson   1      16s

$ kubectl describe secret -n testing-sync-secret regcred
Name:         regcred
Namespace:    testing-sync-secret
Labels:       app.kubernetes.io/managed-by=kyverno
              generate.kyverno.io/clone-policy-name=sync-secrets
              kyverno.io/generated-by-kind=Namespace
              kyverno.io/generated-by-name=testing-sync-secret
              kyverno.io/generated-by-namespace=
              policy.kyverno.io/gr-name=ur-jknht
              policy.kyverno.io/policy-name=sync-secrets
              policy.kyverno.io/synchronize=enable
Annotations:  <none>

Type:  kubernetes.io/dockerconfigjson

Data
====
.dockerconfigjson:  137 bytes
```
As you can see, kyverno automatically sync the secret from `default` namespace to our new created namespace.

There is still many example policy Kyverno, you can check on their official document on [Kyverno](https://kyverno.io/policies/). Or you can create your own policy to harden you Kubernetes Cluster.