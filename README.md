# NVIDIA GLX Desktop Environment Setup on NRP Nautilus (Kubernetes)

These configs help you quickly set up [NVIDIA GLX desktop environments](https://github.com/selkies-project/docker-nvidia-glx-desktop) using Kubernetes on NRP Nautilus. The current configuration is designed to work with the Dog RL project in UCSC's Hare Lab, but you can adapt it to work with your workflow.

With these files, you can:

- Control your ingress to manage multiple desktop deployments.
- Manage resource usage for each desktop deployment.
- Easily start and stop desktop deployments.

Right now, the setup lets you run multiple desktops at once. Each deployment has its own service but shares one ingress. Each deployment only allows one pod per user, so no two users accidentally log into the same desktop.

The current setup includes four deployments per ingress. If you need more, just update the ingress file or create another ingress.

## Quick Start

### Setting up Ingress and Services

To get started, duplicate the `ingress/ingress-template.yaml` file, rename it to something you like, and configure your services as needed — the comments in the config should help guide you through on how to do that.

Next, you can deploy your ingress and services by running:

```bash
kubectl apply -f ingress/<your-custom-ingress>.yaml
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

In the same fashion as the ingress and services, duplicate the `deployments/xgl-deployment-template.yaml` file, rename it to something you like, and configure your deployment and pod template as needed. Again, the comments in there should help guide you through on how to do that.

Now, you can launch a new desktop by deploying your deployment:

```bash
kubectl apply -f deployments/<your-custom-deployment>.yaml
```

Check deployment status:

```bash
kubectl get deploy
kubectl get pods

# More details:
kubectl describe deploy <deployment-name>
kubectl describe pod <pod-name>
```

At this point, the deployment should automatically spin up a new pod, which will be running the Nvidia GLX Desktop Environment along with the Selkies-GStreamer service for streaming. When the pod eventually boots up and shows a status of `[running]`, the service that you created earlier should automatically be able to find the matching pod and connect an endpoint to it. Now, you should be able to go to the host that you configured in your ingress and start using your desktop.

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

This fixes most weird connection problems. If for some reason the stream is still giving you a connection error message (which is common after AFKing for too long or your network hiccups at a bad time), it's most likely Nautilus' TURN server holding onto stale sessions without knowing that they expired. If this happens, delete all deployments and pods, and wait a full 10 minutes before re-deploying and attempting to connect again.

## References

- NRP Nautilus Docs: https://portal.nrp.ai/documentation/userdocs/running/gui-desktop/
- Docker Nvidia GLX Desktop: https://github.com/selkies-project/docker-nvidia-glx-desktop
- Selkies GStreamer
  - https://github.com/selkies-project/selkies
  - https://github.com/selkies-project/selkies/blob/main/docs/firewall.md#turn-server
    – https://datatracker.ietf.org/doc/html/rfc5766section-14.2
    – https://help.hcl-software.com/sametime/11.6/admin/turnserver_ubuntu.html
    – https://github.com/selkies-project/selkies/blob/main/docs/component.md
    – https://nrp.ai/documentation/userdocs/running/gui-desktop/
    – https://gstreamer.freedesktop.org/documentation/webrtc/?gi-language=c
    – https://www.reddit.com/r/WebRTC/comments/1ii3wwm/ice_connection_gets_cancelled_just_after_10
