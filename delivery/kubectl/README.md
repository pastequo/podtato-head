# Deliver with kubectl

Here's how to deliver podtato-head using kubectl.

`kubectl` creates resources from manifests specified via the `--filename/-f`
parameter.

## Deliver

To deploy the manifest in this repo directly with kubectl:

```
kubectl apply -f ./delivery/kubectl/manifest.yaml
```

## Test

Verify that images were retrieved and pods started successfully:

```
kubectl get pods --namespace podtato-kubectl
```

### Test the API endpoint

If using a ClusterIP-type service, run `kubectl port-forward` in the background
and connect through that:

> NOTE: Find and kill the port-forward process afterwards using `ps` and `kill`.

```
# Choose below the IP address of your machine you want to use to access application 
ADDR=0.0.0.0
# Choose below the port of your machine you want to use to access application 
PORT=9000
kubectl port-forward --namespace podtato-kubectl --address ${ADDR} svc/podtato-main ${PORT}:9000 &
```

Now test the API itself with a browser at `http://<uid>.int.be.continental.cloud:9000/`

## Purge

To remove all provisioned resources:

```
kubectl delete -f ./delivery/kubectl/manifest.yaml
```
