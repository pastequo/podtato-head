# Deliver with Kustomize

Here's how to deliver podtato-head using kustomize.

See this guide for info on kustomize: <https://kubectl.docs.kubernetes.io/guides/introduction/kustomize/>.

In short, a set of base resources described in YAML manifests is transformed
("kustomized") in several possible ways before being applied to a cluster. For
example, common annotations can be added to every resource; and image names and
tags can be replaced.

## Deliver

The base resources are described in the directory `delivery/kustomize/base`.

First, preview rendered templates from this base with this command: `kustomize build ./delivery/kustomize/base`

Now, apply the rendered base: `kustomize build ./delivery/kustomize/base | kubectl apply -f -`.

**WARNING** : Kutomize is included in the default kubectl binary under the `-k` option.
So, in theory, you could also apply a kustomization with kubectl like this :
`kubectl apply -k ./delivery/kustomize/base`
_BUT_ it often leads to errors since the embedded version is not following kustomize official releases.
So please prefer the standalone `kustomize` CLI which is safer than the embedded `-k` one.

### Deliver an overlay

kustomize "overlays" transform resources from the original base. An overlay can
point to any other overlay or base as its own base.

Look in `delivery/kustomize/overlay` for an example of an overlay which
transforms resource for delivery to a production environment by adding labels
and modifying image names.

Render the overlay with `kustomize build ./delivery/kustomize/overlay`

Apply it with `kustomize build ./delivery/kustomize/overlay | kubectl apply -f -`

You may use commands like the following to generate a diff from base to overlay and verify it meets expectations:

```bash
kustomize build ./delivery/kustomize/base > rendered_base.yaml
kustomize build ./delivery/kustomize/overlay > rendered_overlay.yaml
diff rendered_base.yaml rendered_overlay.yaml
```

## Test

Check for resources for the base in the current context namespace: `kubectl get pods`.

Check for resources for the overlay in the `podtato-kustomize-production` namespace: `kubectl get pods --namespace podtato-kustomize-production`.

### Test the API endpoint

If using a ClusterIP-type service, run `kubectl port-forward` in the background
and connect through that:

> NOTE: Find and kill the port-forward process afterwards using `ps` and `kill`.

```
# Choose below the IP address of your machine you want to use to access application 
ADDR=0.0.0.0
# Choose below the port of your machine you want to use to access application 
PORT=9000
kubectl port-forward --address ${ADDR} svc/podtato-main ${PORT}:9000 &
```

Now test the API itself with a browser at `http://<uid>.int.be.continental.cloud:9000/`

## Purge

```
kustomize build ./delivery/kustomize/base | kubectl delete -f -
kustomize build ./delivery/kustomize/overlay | kubectl delete -f -
```