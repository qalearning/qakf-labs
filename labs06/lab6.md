# Lab 6 - Security
## 6.1 RBAC

We're going to create a pod that prints all of the logs of all of our randoms jobs.

Recreate the Job from the previous lab (with 10 completions)

Create a pod using the kubectl image. Give it a command property that runs the bash command that we came up with in the previous lab to print all jobs pods logs.

Create the pod. It should fail.

Create a clusterrole that allows  get, list pods and pods/logs

Create a rolebinding to bind that role to the default sa in the default ns

Delete and recreate the logger pod.

## 6.2 NetPol

curl the frontend service and the backend service in each ns.

Create a netpol that allows traffic on port 80 to pods with an app label with a value of frontend.

Apply it in both (/ all 3 ns)

Again, curl the frontend service and the backend service in each ns.

Create another netpol that allows traffic to pods with an app label with a value of backend from pods with an app label of frontend from a namespace with a name label of prod. Apply it to the prod namespace.

Create another netpol as above for the dev namespace.

Try curling both the frontend and backend services again.

Make sure that the ingresses from an earlier lab still work

(stretch: all 3 ns)

## 6.3 Pod Security

Create an httpd pod with a securityContext that says runAsNonRoot: true.

It won't work.

Add a runAsNonRoot: true to all of your frontend deployments. You may need to recreate them.

They should be fine, because they're both listening on port 8080 and k8s can tell that they don't need to run as root.

(stretch: what about the backends? **answer** author isn't sure... (maybe runasuser: 1000? Try it!))

That's it, you're done! Let your instructor know that you've finished the lab.