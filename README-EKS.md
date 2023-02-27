## Installing nameko-dev on EKS
### 
* Pre-requisites: Install Helm and eksctl
- Commands to install Helm:
```curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
   chmod 700 get_helm.sh
   ./get_helm.sh
```
- Install or upgrade eksctl. If eksctl is already installed, the following command upgrades and relinks it. Or if eksctl isn't installed yet, the following command installs Weaveworks Homebrew tap as needed, and then installs eksctl:
```brew upgrade eksctl && { brew link --overwrite eksctl; } || { brew tap weaveworks/tap; brew install weaveworks/tap/eksctl; }
```
* Create EKS Cluster using ekctl:
```eksctl create cluster \
  --name=epinio \
  --region=us-west-1 \
  --nodes=2 \
  --node-type=t3.xlarge \
  --node-volume-size=40 \
  --managed \
  --kubeconfig=kubeconfig-eks
```
- To avoid problems in your PVC:
```eksctl get addon --name aws-ebs-csi-driver --cluster epinio --region=us-west-1 
```
- To access your Cluster you can get your kube config file to use in Lens, for example:
```aws eks --region=us-west-1 example_region update-kubeconfig --name epinio
```
* Install Certmanager
```helm repo add cert-manager https://charts.jetstack.io
helm repo update
helm install cert-manager --namespace cert-manager --create-namespace jetstack/cert-manager --set installCRDs=true --set extraArgs={--enable-certificate-owner-ref=true}
```

* Install Ngnix Ingress Controller
- Add Helm repo
```helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
   helm repo update
```
- Install
```helm upgrade --install nginx ingress-nginx/ingress-nginx --namespace nginx --create-namespace --set controller.ingressClassResource.default=true
```
* Create a CNAME DNS entry for your ELB
- The ELB endpoint is automatically assigned after installing ingress-nginx-controller. For getting the assigned ELB endpoint in your cluster run this command:
```kubectl get svc -n nginx nginx-ingress-nginx-controller -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```
Use that ELB endpoint value when creating the CNAME record for your DNS zone (for eg. in AWS Route53 service).

* Install Epinio
```helm upgrade --install epinio epinio/epinio --namespace epinio --create-namespace --set global.domain=giselle.eng.br --set global.tlsIssuer=letsencrypt-production --set global.tlsIssuerEmail=email@example.com
```
