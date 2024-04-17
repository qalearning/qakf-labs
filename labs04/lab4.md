# Lab 4 - Networking
## 4.1 Explore CoreDNS

1. Create a `public.ecr.aws/w4e1v2x6/qa-wfl/qakf/sbe` deployment in each of the `dev` and `prod` namespaces, using the `:v2` image in `dev` and the `:v1` image in `production`. Expose them as `clusterIP` services on port 8080.

<details><summary>show commands</summary>
<p>

```bash
kubectl create deploy lab4backend --image=public.ecr.aws/w4e1v2x6/qa-wfl/qakf/sbe:v1 -n production 
kubectl create deploy lab4backend --image=public.ecr.aws/w4e1v2x6/qa-wfl/qakf/sbe:v2 -n development
```

</p>
</details>
<br/>

2. Expose them both as `clusterIP` services on port 8080, giving them both a `name` of `backend`.

<details><summary>show command</summary>
<p>

```bash
kubectl expose deployment lab4backend --port 8080 --name backend --namespace production 
kubectl expose deployment lab4backend --port 8080 --name backend -n development
```

</p>
</details>
<br/>

3. We're going to use the `busybox` image to interact with DNS using nslookups. Create a pod named `nettools` in both the `dev` and `prod` namespaces. Use the `busybox` image. You'll need it to run a `command` of `sleep infinity` or it will immediately tranistion to a `completed` state.

<details><summary>show commands</summary>
<p>

```bash
kubectl run nettools --image=busybox -n production --command sleep infinity
kubectl run nettools --image=busybox -n development --command sleep infinity
```

</p>
</details>
<br/>

4. Use `kubectl exec` to execute the command `nslookup backend` on the two pods in the two different namespaces.

<details><summary>show commands</summary>
<p>

```bash
kubectl exec -it nettools -n production -- nslookup backend
kubectl exec -it nettools -n development -- nslookup backend
```

</p>
</details>
<br/>


Example output:

```
Server:         10.96.0.10
Address:        10.96.0.10:53

** server can't find backend.cluster.local: NXDOMAIN

Name:   backend.development.svc.cluster.local
Address: 10.107.29.209

** server can't find backend.svc.cluster.local: NXDOMAIN

** server can't find backend.cluster.local: NXDOMAIN


** server can't find backend.svc.cluster.local: NXDOMAIN

command terminated with exit code 1
```
<br/>

Note that CoreDNS tried a lot of variations of the name `backend` but one of the first was for `backend.development.svc.clsuter.local` in this case. When run against the prod namespace, it'll look there. This is to illustrate that CoreDNS "knows" which namespace a pod is running in and returns the appropriate lookup.

5. There's only one placeholder remaining in our simple front end application, for the data from the backing service. Let's finish that off now by `apply`ing (a copy of) the sfe deployment in both namespaces. Again, you might wish to change the `lab3frontend`s to `lab4frontend`s or simply `frontend`s.

<details><summary>show command</summary>
<p>

```bash
kubectl apply -n production -f lab4frontend.yaml
kubectl apply -n development -f lab4frontend.yaml
```

</p>
</details>
<br/>

6. Get the new pods' details either from the dev or the prod namespaces. It doesn't matter which. Ask for `wide` output so you can see their IP addresses.

<details><summary>show command</summary>
<p>

```bash
kubectl -n production get pods --output wide
```

</p>
</details>
<br/>

Example output:

```bash
TBD
```

<br/>

7. **cURL** one of the pods IP addresses. You should see a v2 message in the dev namespace and a v1 message in production.

8. **Optional but maybe interesting** `exec` into one of your pods and run `cat /code/app/main.py`. See (around the 25th line) it's just asking for "http://backend"? That's basically what you did earlier with the nslookups. CoreDNS still knows which namespace your workload is running in.



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
