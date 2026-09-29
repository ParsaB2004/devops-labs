# Lab: Deployment with 3 replicas

A Deployment keeps the desired number of Pods running.

## Apply

```bash
kubectl apply -f deployment.yaml
kubectl get pods
```

Expected: three Pods named `hello-kube-<hash>-<id>` in `1/1 Running`.

## Self-healing

Delete one Pod and list again:

```bash
kubectl delete pod <pod-name>
kubectl get pods
```

A new Pod is created immediately and the count returns to three.

## Clean up

```bash
kubectl delete -f deployment.yaml
```
