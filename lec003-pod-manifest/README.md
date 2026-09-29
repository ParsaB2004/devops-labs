# Lab: Declarative Pod on kind with a private registry

Run a Pod from a YAML manifest on a local kind cluster, pulling the image from the private registry on `localhost:5000`.

## Create the cluster

`kind-config.yaml` enables containerd's per-registry config directory:

```bash
kind create cluster --name hello --config kind-config.yaml
```

## Connect the registry to the cluster

kind nodes run inside containers, so `localhost:5000` from a node is the node itself. Attach the registry to the `kind` network and map the name inside the node:

```bash
docker network connect kind registry

REGISTRY_DIR="/etc/containerd/certs.d/localhost:5000"
docker exec hello-control-plane mkdir -p "${REGISTRY_DIR}"
cat <<EOT | docker exec -i hello-control-plane cp /dev/stdin "${REGISTRY_DIR}/hosts.toml"
[host."http://registry:5000"]
EOT
```

## Apply the manifest

```bash
kubectl apply -f pod.yaml
kubectl get pods
kubectl port-forward pod/hello-kube 8080:80
```

Open http://localhost:8080. Applying again prints `pod/hello-kube unchanged`, which is the declarative behavior: the cluster already matches the desired state.

## Clean up

```bash
kubectl delete -f pod.yaml
```

## What I learned

- All containers in a Pod share one network namespace. Three nginx containers in one Pod fail with `bind() to 0.0.0.0:80 failed (98: Address in use)`, so replicas belong in a Deployment, not in extra containers.
