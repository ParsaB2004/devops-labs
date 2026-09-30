# Lab: Imperative Deployment and labels

Create a Deployment with a single `kubectl` command, inspect the labels it puts on its Pods, reach a Pod through the Deployment, and clean up. The image comes from the private registry on `localhost:5000` (see [lec002](../lec002-docker-registry) and [lec003](../lec003-pod-manifest) for the setup).

For the declarative version with 3 replicas, see [lec003-deployment](../lec003-deployment).

## Create the Deployment

```bash
kubectl create deployment hello-kube --image=localhost:5000/hello_kube:1.1.0
kubectl get deployments
kubectl get deploy hello-kube
```

Right after creation the Deployment shows `READY 0/1`. After a few seconds it becomes `1/1`:

```
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
hello-kube   1/1     1            1           25s
```

## Inspect labels

```bash
kubectl get deploy hello-kube -o jsonpath="{.spec.template.metadata.labels}"
kubectl get pods -l app=hello-kube
kubectl get pods --show-labels
```

Output:

```
{"app":"hello-kube"}
NAME                          READY   STATUS    RESTARTS   AGE
hello-kube-549b5dd5cd-w6z2s   1/1     Running   0          64s
...   app=hello-kube,pod-template-hash=549b5dd5cd
```

The Deployment stamps `app=hello-kube` on every Pod it creates, and its selector uses that label to find the Pods it owns. The extra `pod-template-hash` label is added by the ReplicaSet that sits between the Deployment and its Pods.

## Access a Pod through the Deployment

```bash
kubectl port-forward deploy/hello-kube 8080:80
```

Open http://localhost:8080. kubectl picks one Pod of the Deployment and forwards traffic to port 80 of that Pod.

## Clean up

```bash
kubectl delete deploy hello-kube
kubectl get deployments
kubectl get pods
```

Deleting the Deployment also removes its Pods. Both `get` commands return `No resources found in default namespace.`

## What I learned

- A Deployment manages Pods through labels and selectors, not by name.
- `kubectl create deployment` is imperative: fast for experiments, but the result is not recorded anywhere. The YAML approach in lec003 is reproducible and can be reviewed in git.
- Pod names follow the pattern `<deployment>-<replicaset-hash>-<id>`, which shows the Deployment -> ReplicaSet -> Pod chain.

## Troubleshooting note

After a reboot, every `kubectl` command failed with `x509: certificate has expired or is not yet valid`. The system clock was correct, but the kind cluster's API server certificate had a `notBefore` about 3 hours in the future (checked with `openssl x509 -noout -dates` inside the node). Most likely the cluster was created while the clock was off. Deleting and recreating the cluster fixed it:

```bash
kind delete cluster --name hello
kind create cluster --name hello --config ../lec003-pod-manifest/kind-config.yaml
```

The images survived because they live in the registry's named volume, not in the cluster. After recreating the cluster, the registry hookup from the lec003 README has to be repeated.
