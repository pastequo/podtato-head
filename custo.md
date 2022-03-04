# Custo step

## Setup

1. Replace LoadBalancer by ClusterIP

`find delivery -type f -name "*.yaml" -print0 | xargs -0 sed -i 's/type: LoadBalancer/type: ClusterIP/g'`

`find delivery -type f -name "*.yaml" -print0 | xargs -0 sed -i 's/serviceType: LoadBalancer/serviceType: ClusterIP/g'`

2. Install kind

`go install sigs.k8s.io/kind@v0.11.1`

3. Create kind cluster

`kind create cluster`

## Kubectl

`kubectl apply -f ./delivery/kubectl/manifest.yaml`

`kubectl port-forward svc/podtato-main 9000:9000`

`firefox http://localhost:9000`

Edit some image, apply, reload

`kubectl delete -f ./delivery/kubectl/manifest.yaml`

## Helm

`kubectl create namespace podtato-kubectl`

`helm install podtato-head ./delivery/chart`

`kubectl port-forward svc/podtato-main 9000:9000`

`firefox http://localhost:9000`

Edit some images

`helm upgrade --install podtato-head ./delivery/chart`

reload

`helm history podtato-head`

`helm rollback podtato-head 1`

reload

`helm status podtato-head`

`helm uninstall podtato-head`

## Kustomize

`kstomize build ./delivery/kustomize/overlay/prod | kubectl apply -f -`

(kstomize edit set namespace)

`kubectl port-forward svc/podtato-main 9000:9000`

`firefox http://localhost:9000`

`kstomize build ./delivery/kustomize/overlay/prod | kubectl delete -f -`

Discourage the use via kubectl -k

## Kapp

`export PATH=$PWD/kapp-install/:$PATH`

`kns default`

Prepare `kubectl get po -n podtato-kubectl`

`kapp deploy -a podtatoserver-app -f ./delivery/kubectl/manifest.yaml`

"app" created in default namespace

`kapp list`

`kapp inspect -a podtatoserver-app --tree`

`kns podtato-kubectl`

`kapp list`

`kubectl port-forward svc/podtato-main 9000:9000`

`firefox http://localhost:9000`

`kns default`

`kapp deploy -a podtatoserver-app -f ./delivery/kubectl/manifest.yaml --diff-changes`

`kapp delete -a podtatoserver-app`

## ArgoCD

Install in cluster:

`kubectl create namespace argocd`
`kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml`

Check it's started

`kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo`

`argocd login --insecure --grpc-web --port-forward --port-forward-namespace argocd`

username: admin
password: previous command

`argocd account update-password --insecure --grpc-web --port-forward --port-forward-namespace argocd`

`kubectl -n argocd port-forward --address 0.0.0.0 svc/argocd-server 8082:80`

`firefox https://localhost:8082`

Create app, path `delivery/kubectl`, choose directory, sync policy automatic

Show UI

Do commit

