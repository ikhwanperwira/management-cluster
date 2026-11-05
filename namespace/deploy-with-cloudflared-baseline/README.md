# Deploy with Cloudflared baseline

This is a minimal private HTTP origin for validating the Cloudflare Tunnel used
by `codex-system`. Configure a tunnel public hostname to target:

```text
http://hello-world.deploy-with-cloudflared-baseline.svc.cluster.local:80
```

The origin remains a `ClusterIP` Service; the existing `codex-system/cloudflared`
Deployment provides the tunnel connection.
