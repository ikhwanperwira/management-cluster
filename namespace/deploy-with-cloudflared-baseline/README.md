# Deploy with Cloudflared baseline

This is a minimal HTTP origin for validating a Cloudflare Tunnel public
hostname. Configure the subdomain you want to use in the tunnel ingress rules
and target this origin:

```text
http://hello-world.deploy-with-cloudflared-baseline.svc.cluster.local:80
```

The origin remains a private `ClusterIP` Service. The tunnel connector and its
credentials are managed outside this namespace.
