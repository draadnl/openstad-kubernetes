## cert-manager

### Backup
```kubectl get --all-namespaces -oyaml issuer,clusterissuer,cert > backup.yaml\n```

### Upgrade to v1.2.0
```helm upgrade --set installCRDs=true --version=v1.2.0 cert-manager jetstack/cert-manager -n cert-manager```


## ingress-nginx

### Risky!
**Uninstalling `ingress-nginx` will clear the given IP for the load balancer. Installing a new version will likely result in a new IP.**

### Create values.yaml
```yaml
controller:
  service:
    externalTrafficPolicy: Local
    loadBalancerIP: 178.128.141.36
  watchIngressWithoutClass: true
rbac:
  create: true
```

### uninstall
```helm uninstall nginx-ingress -n default```


### install
```helm install ingress-nginx ingress-nginx --values=values.yaml --repo https://kubernetes.github.io/ingress-nginx --namespace default```


### update configmap `ingress-nginx-controller`

Compare with these values:
```yaml
  allow-snippet-annotations: "true"
  compute-full-forwarded-for: "true"
  enable-modsecurity: "true"
  enable-owasp-modsecurity-crs: "true"
  proxy-real-ip-cidr: 10.0.0.0/8
  real-ip-header: proxy_protocol
  server-tokens: "false"
  use-forwarded-headers: "true"
```

## argocd

### upgrade to v2.3.3
```k -n argocd apply -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.3.3/manifests/install.yaml```


## Issues

### `openstad-<namespace>-mysql-master` Stateful set
ArgoCD shows this STS as trying to remove its apiVersion from the `volumeClaimTemplates`:

```yaml
    - apiVersion: v1
      kind: PersistentVolumeClaim
      metadata:
```

This is not allowed, so should not happen

### Ingress files
All ingress files still show up as the old api version in `kubent` — even though they are migrated to the new version — because of the `kubectl.kubernetes.io/last-applied-configuration:` annotation still containing the old v1beta1 apiVersion.
