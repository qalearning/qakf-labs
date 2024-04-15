# Lab 3 - Volumes and Data
## 1.1 Volume mounts

Create an apache deployment, but mount index.html. You'll need to create index.html on both worker nodes (for now - this is where a persistent volume might come in handy)

## 1.2 ConfigMaps

create a configmap from index.html.
You can now delete the files from the worker nodes.

Create a configmap in both namespaces with a different colour in each.
Add a configmap to the podspecs for the sfe deployments.
Create a sfe deployment in both ns
Expose the two deployments.
Browse to them.
(stretch: test ns too)

## 1.3 Secrets

Create a secret called secrets with a key of password in the dev and prod namespaces, with different values for each.
Add a volume mount to the deployment and recreate them.
Browse to the sfe deployments.
(stretch: test ns too)
