# Lab: Declarative Deployment with 3 replicas

Describe a Deployment in a YAML manifest and manage it with `kubectl apply` and `kubectl delete`. The image comes from the private registry on `localhost:5000` (see [lec002](../lec002-docker-registry) and [lec003](../lec003-pod-manifest) for the setup).

For the imperative version, see [lec004-deployment-imperative](../lec004-deployment-imperative).

## The manifest

`deployment.yaml` defines a Deployment named `hello-kube` with three replicas. The `selector` and the Pod template labels must match (`app: hello-kube`); that label is how the Deployment finds the Pods it owns.

The original exercise manifest has no `replicas` field (default: 1) and uses a Docker Hub image. Here I set `replicas: 3` and use the image from my private registry.

## Apply

```bash
kubectl apply -f deployment.yaml
kubectl get deployments
kubectl get pods
```

Expected after a few seconds: `READY 3/3` and three Pods named `hello-kube-<hash>-<id>` in `1/1 Running`. Running `get` in the first second shows `0/3` and `ContainerCreating`, which is normal.

## Declarative behavior

Applying the same manifest twice shows the difference from imperative commands:

```bash
kubectl apply -f deployment.yaml
kubectl apply -f deployment.yaml
```

```
deployment.apps/hello-kube created
deployment.apps/hello-kube unchanged
```

The second run changes nothing because the cluster already matches the desired state.

## Self-healing

Delete one Pod and list again:

```bash
kubectl delete pod <pod-name>
kubectl get pods
```

A replacement Pod is created immediately and the count returns to three.

## Clean up

The manifest is also the source of truth for removal:

```bash
kubectl delete -f deployment.yaml
kubectl get pods
```

The Deployment and all three Pods are removed. The Pods show `Terminating` briefly, then `No resources found in default namespace.`

## What I learned

- `apply` is idempotent: repeating it is safe and reports `unchanged` when nothing differs.
- The manifest lives in git, so the same cluster state can be recreated anywhere, unlike a one-off `kubectl create`.
- Replica count is part of the desired state; the Deployment controller keeps reconciling the cluster toward it.
