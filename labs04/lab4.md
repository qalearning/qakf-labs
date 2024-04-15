# Lab 4 - Networking
## 4.1 Explore CoreDNS
create and expose an sbe deployment in each ns (as a clusterIP) (different versions)
spin up a busybox in each ns.
exec an nslookup against each one looking for the "backend" service.
create sfe deployments if necessary.
exec into one of the pods and cat /code/app/main.py. See it's just asking for "http://backend"?
browse to the two deployments

## 4.2 Install an ingress controller

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.0/deploy/static/provider/cloud/deploy.yaml

check it's there

kubectl get svc --all-namespaces

browse to yourIp:nodePort. You should get an error because we haven't configured any backends.

## 4.3 Expose the frontends

in both / all 3 ns. Make sure you use a clusterIP.
curl them all.
create an ingress for dev using nip.io in the dev ns
point you browser at dev.your-ip.nip.io:nodePort
create an ingress for prod in the prod ns
test it
(stretch: and test ns)
**Needs checking**
