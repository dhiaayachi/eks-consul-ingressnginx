# Consul on EKS using nginx as ingress (transparent mode)

Deploy an EKS cluster:

```bash
eksctl create cluster --name=<cluster name> --region=<region> --nodes=3 --version 1.21
```

Import kubectl config

```bash
aws eks update-kubeconfig --region <region> --name <cluster name>
```

Deploy Consul

```bash
helm install consul hashicorp/consul --values consul-values.yaml --create-namespace --namespace consul
```

Add deny all intention
```bash
kubectl apply -f denyall.yaml
```

Deploy nginx ingress
```bash
helm upgrade --install ingress-nginx ingress-nginx \                                                                                                    ─╯
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace --values nginxingress-values.yaml
```

Add configuration for Dialed Directly

```bash
kubectl apply -f sd-direct.yaml
```

Add configuration for ingress config to static-server. If there are issues with validating the ingress object see https://github.com/kubernetes/ingress-nginx/issues/5968 for workarounds. 
```bash
kubectl apply -f ingress.yaml
```

deploy static-server
```bash
kubectl apply -f static-server.yaml
```

add intention to static server from ingress
```bash
kubectl apply -f allow-static-server.yaml
```

To get the ingress endpoint provisioned in ELB:
```bash
kubectl get service  -n ingress-nginx
```

Ensure you get back a `hello world` response:

```bash
STATIC_SERVER_LB=$(kubectl get service  -n ingress-nginx ingress-nginx-controller -o jsonpath="{.status.loadBalancer.ingress[*].hostname}")
curl $STATIC_SERVER_LB
```

