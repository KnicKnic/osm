# Quick Demo
Here is a quick demo of setting up the nginx ingress controller inside the OSM service mesh. It allows the nginx ingress controller to have a public endpoint (port 80) to be exposed, and then forwards traffic using ingresses & the service mesh.

## presetup
Setup osm injection & enable help charts
```bash
# build osm
make build-osm 
#install osm
./bin/osm install --set=OpenServiceMesh.image.tag=592240ab08bc401decd3f1d39ed326a436744610


# enable OSM in the default namespace
# *note I disable sidecar injection
./bin/osm namespace add default --disable-sidecar-injection
# Namespace [default] successfully added to mesh [osm]

# install helm
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

# install helm with sidecar injection, and exclude port 80 inbound
# also I disabled controller admission hooks
helm upgrade --install test ingress-nginx/ingress-nginx --set controller.ingressClass=test --set-string controller.podAnnotations.'openservicemesh\.io/sidecar-injection'="enabled"  --set-string controller.podAnnotations.'openservicemesh\.io/inbound-port-exclusion-list'="80"  --set controller.admissionWebhooks.enabled=false
```


## Files
* mesh config - [meshconfig-set.yaml](./meshconfig-set.yaml)
* pods & traffic rules - [nginx.yaml](./nginx.yaml)


## Demo / commands to restart
```bash
# delete every thing to ensure that it is setup for this test
kubectl delete ingress nginx
kubectl apply -f ./nginx.yaml
kubectl apply -f ./meshconfig-set.yaml

# restart everything in order to ensure that new values are set
kubectl rollout restart -n osm-system deploy/osm-controller
kubectl rollout status -n osm-system deploy/osm-controller

kubectl rollout restart deploy/test-ingress-nginx-controller
kubectl rollout restart deploy/nginx-deployment

kubectl rollout status deploy/test-ingress-nginx-controller
kubectl rollout status deploy/nginx-deployment
```

## test if things work
```bash
curl --verbose http://myapi.mydomain.com --resolve myapi.mydomain.com:80:<!!!!!Substitute your ingress IP HERE!!!!!>
kubectl exec -it deploy/test-ingress-nginx-controller -- curl --verbose http://myapi.mydomain.com --resolve myapi.mydomain.com:80:127.0.0.1
```