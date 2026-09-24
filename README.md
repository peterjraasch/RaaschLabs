# RaaschLabs

GitOps repo for the single-node k3s platform. Adding `apps/<name>/values.yaml` and pushing makes
`https://<name>.raaschlabs.com` live.

- `platform/` - cluster issuer, wildcard cert, Argo CD ingress, ApplicationSet (applied by hand once)
- `charts/spring-app/` - reusable chart; release name = subdomain
- `apps/` - one folder per app (values only)

Sibling projects (separate repos): `../www` (directory of running apps), `../resume` (peter.raaschlabs.com).

## Claim the domain

1. Register `raaschlabs.com` at Cloudflare Registrar (at-cost pricing, and DNS is already where the plan needs it). Route 53 or any registrar also works if you then point the nameservers at Cloudflare.
2. Confirm the zone is active in Cloudflare, then create the API token (Zone > DNS > Edit, Zone > Zone > Read) scoped to it.
3. Follow Phases 1-5 of the plan (EC2, k3s, wildcard DNS, cert-manager, Argo CD).

## Adding an app

Create `apps/<name>/values.yaml`, push. Useful values:

```yaml
image: ghcr.io/peterjraasch/<repo>:<sha>
imagePullSecrets: [{ name: ghcr }]
title: Recipes                 # shown on www.raaschlabs.com
description: One-line summary  # shown on www.raaschlabs.com
listed: true                   # false hides it from the directory
```

The `www` app reads Ingress annotations from the `apps` namespace, so new apps appear automatically.

## Apps to add once their images are built

`apps/www/values.yaml` (also serves the bare `raaschlabs.com`):

```yaml
image: ghcr.io/peterjraasch/www:<sha>
imagePullSecrets: [{ name: ghcr }]
title: Raasch Labs
description: Directory of apps running on this platform.
extraHosts: [raaschlabs.com]
rbac: { readIngresses: true }
listed: false
```

`apps/peter/values.yaml`:

```yaml
image: ghcr.io/peterjraasch/resume:<sha>
imagePullSecrets: [{ name: ghcr }]
title: Resume
description: Peter Raasch's resume.
```

Do not add these before the images exist; Argo CD would deploy pods stuck in `ImagePullBackOff`.
