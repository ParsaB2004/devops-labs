# DevOps Labs

Hands-on DevOps exercises I built while following a DevOps course. Each folder is a self-contained lab with its own README, commands, and expected output.

**Author:** Parsa Behjati

## Labs

| Lab | Topic | Tools |
|-----|-------|-------|
| [lec002-docker-registry](lec002-docker-registry) | Containerized nginx page and a private Docker registry | Docker, nginx, registry |
| [lec003-pod-manifest](lec003-pod-manifest) | Declarative Pod on a local kind cluster pulling from the private registry | Kubernetes, kind, kubectl |
| [lec003-deployment](lec003-deployment) | Deployment with 3 replicas and self-healing | Kubernetes, kubectl |
| [lec004-deployment-imperative](lec004-deployment-imperative) | Imperative Deployment, labels and selectors, port-forward to a Deployment | Kubernetes, kubectl |

## Environment

- Ubuntu Linux
- Docker
- kind v0.31.0 (Kubernetes v1.35.0)
- kubectl v1.32.0

## Notes

Images are pushed to a local registry at `localhost:5000`, so the manifests in this repo assume that registry exists. Setup steps are in the [lec002 README](lec002-docker-registry/README.md).
