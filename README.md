# NVIDIA GLX Desktop Environment Setup on NRP Nautilus (Kubernetes)

These configs help you quickly set up [NVIDIA GLX desktop environments](https://github.com/selkies-project/docker-nvidia-glx-desktop) using Kubernetes on NRP Nautilus.

With these files, you can:

- Control your ingress to manage multiple desktop deployments.
- Manage resource usage for each desktop deployment.
- Easily start and stop desktop deployments.

Right now, the setup lets you run multiple desktops at once. Each deployment has its own service but shares one ingress. Each deployment only allows one pod per user, so no two users accidentally log into the same desktop.

The current setup includes four deployments per ingress. If you need more, just update the ingress file or create another ingress.

## Quick Start

### Setting up Ingress and Services

To get started, run:

```bash
kubectl apply -f ingress/xgl-ingress.yaml
```

Check if everything worked:

```bash
kubectl get ingress
kubectl get svc

# If you want details:
kubectl describe ingress <ingress-name>
kubectl describe svc <service-name>
```

Services will initially show no active endpoints since no deployments are running yet. Also, it's normal if services don’t have external IPs—the ingress handles external traffic.

### Creating a Desktop Deployment

Launch a new desktop:

```bash
kubectl apply -f deployments/xgl-deploy-main.yaml
```

Check deployment status:

```bash
kubectl get deploy
kubectl get pods

# More details:
kubectl describe deploy <deployment-name>
kubectl describe pod <pod-name>
```

If something looks wrong, check pod logs:

```bash
kubectl logs <pod-name>
```

If the pod is up, check if the service connected to it:

```bash
kubectl describe svc <service-name>
```

You can now connect through your ingress URL!

### Stopping a Desktop Deployment

To delete a desktop:

```bash
kubectl get deploy
kubectl delete deploy <deployment-name>
```

This removes the deployment and shuts down the pod. The service stays available for future deployments. Check the service with:

```bash
kubectl get svc
```

## Troubleshooting

If your client connects but immediately disconnects after "waiting for stream," usually something's off with ingress or services pointing to dead pods.

Check details with:

```bash
kubectl describe svc <service-name>
kubectl describe ingress <ingress-name>
```

If nothing obvious shows up, deleting and recreating usually helps:

```bash
kubectl delete svc <service-name>
kubectl delete ingress <ingress-name>

kubectl apply -f ingress/xgl-ingress.yaml
```

This fixes most weird connection problems.

## References

- NRP Nautilus Docs: https://portal.nrp.ai/documentation/userdocs/running/gui-desktop/
- Docker Nvidia GLX Desktop: https://github.com/selkies-project/docker-nvidia-glx-desktop
- Selkies GStreamer
  - https://github.com/selkies-project/selkies
  - https://github.com/selkies-project/selkies/blob/main/docs/firewall.md#turn-server
