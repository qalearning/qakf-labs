# Lab 3 - Volumes and Data
## 3.1 Volume mounts

### Task 1 - explore emptyDirs

1. Create a pod manifest using the `abhirockzz/kvstore` image use the `dry-run` and `-o yaml` technique to create a file called `lab3kv.yaml`. This is an image that runs a simple key/value store that uses the file system to store values.

<details><summary>show command</summary>
<p>

```bash
kubectl run kvstore --image=abhirockzz/kvstore --dry-run=client -o yaml > lab3kv.yaml
```

</p>
</details>
<br/>

2. Apply the YAMLfest and then find the new pod's IP address.

<details><summary>show command</summary>
<p>

```bash
kubectl apply -f lab3kv.yaml
kubectl get pod -o wide
```

</p>
</details>
<br/>

Example output:

```bash
pod/kvstore created

NAME  READY  STATUS   RESTARTS   AGE     IP               NODE
test  1/1    Running  0          22m     192.168.230.21   k8s-worker-1
```

<br/>

3. Try to put some data into the store. It will fail.

```bash
 curl 192.168.230.21:8080/save -d 'name=Nefertiti'
```

Example output:

```bash
Failed to create file /data/name due to --- open /data/name: no such file or directory
```

<br/>

4. We need to ensure that the `/data` directory exists. Start by deleting the existing pod.

<details><summary>show command</summary>
<p>

```bash
kubectl delete pod kvstore
```

</p>
</details>
<br/>

5. Modify the pod specification in the YAMLfest so that it adds a volume named `data-volume` of the `emptyDir` type and adds a volumeMount to the container definition that mounts `data-volume` at `/data`

lab3kv.yaml (partial):

```yaml
spec:
  containers:
  - image: abhirockzz/kvstore
    name: kvstore
    volumeMounts:
    - mountPath: /data
      name: data-volume
  volumes:
    - name: data-volume
      emptyDir: {}
```

6. Apply the YAMLfest again and find out the new pod's IP address.

<details><summary>show command</summary>
<p>

```bash
kubectl apply -f lab3kv.yaml
kubectl get pod -o wide
```

</p>
</details>
<br/>

Example output:

```
pod/kvstore created

NAME  READY  STATUS   RESTARTS   AGE     IP               NODE
test  1/1    Running  0          22m     192.168.230.22   k8s-worker-1
```

<br/>

7. Retry adding data to the store. Make sure you're curling your pod's IP address.

```bash
 curl 192.168.230.22:8080/save -d 'name=Nefertiti'
```

Example output:

```bash
Saved value Nefertiti to /data/name
```

<br/>

8. Feel free to add more key/value pairs and then try reading the data.

```bash
 curl 192.168.230.22:8080/read/name
```

Example output:

```bash
Nefertiti
```

<details><summary>Stretch goal - optional exercise</summary>
<p>

9. **OPTIONAL stretch goal** see if you can find the emptyDir in your hosts' file system. It will involve finding out which node the pod is running on, connecting to that node and working out where in the file system the emptyDir is (you might be able to `find` a file named `data-volume`). Once you have found it, you could look for the files therein. Also, if you do take on this chalenge, observe, once you've deleted the pod, that the directory is removed.

</p>
</details>
<br/>

10. Delete the pod.

### Task 2 - explore hostPaths

11. Create a deployment YAMLfest named `lab3web`, using the `httpd` image with 3 replicas, in a file named `lab3web.yaml`

<details><summary>show command</summary>
<p>

```bash
kubectl create deployment lab3web --image=httpd --replicas=3 --dry-run=client -o yaml > lab3web.yaml
```

</p>
</details>
<br/>

12. `Apply` the YAMLfest and `expose` it as a service on port 80.

<details><summary>show command</summary>
<p>

```bash
kubectl apply -f lab3web.yaml && kubectl expose deployment lab3web --port=80
```

</p>
</details>
<br/>

13. Find out the `ClusterIP` of the new service and **cURL** it.

<details><summary>show command</summary>
<p>

```bash
kubectl get service lab3web
curl ip-from-previous-command
```

</p>
</details>
<br/>

Example output:

```
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
lab3web      ClusterIP   10.43.137.86   <none>        80/TCP           5s

curl 10.43.137.86
```

<br/>

Example output:

```bash
<html><body><h1>It works!</h1></body></html>
```


<br/>

14. Now create a file named `index.html` in your home directory. Its contents should be similar to the following, but feel free to customise the "welcome" message to suit yourself:

```html
<html><body><h1>Welcome to my home page!</h1></body></html>'
```

<details><summary>show command</summary>
<p>

```bash
echo '<html><body><h1>Welcome to my home page!</h1></body></html>' > ~/index.html
```

</p>
</details>
<br/>

15. Delete the deployment (but not the service).

16. Modify the lab3web.yaml file to add a volume of the `hostPath` type, with volume type of `File` and a path of `/home/student/index.html`. Mount the volume in the deployment's pod at a `mountPath` of `/usr/local/apache2/htdocs/`.

<details><summary>show YAML</summary>
<p>

```yaml
        volumeMounts:
        - name: homepage
          mountPath: /usr/local/apache2/htdocs/
      volumes:
      - name: homepage
        hostPath:
          path: /home/student/index.html
```

</p>
</details>
<br/>

17. **cURL** the service's IP address again.

```bash
curl your-service-ip-address
```

Example output:

```html
curl: (7) Failed to connect to 10.101.251.165 port 80 after 0 ms: Connection refused
```

<br/>

That's not your home page! The issue is that the file `/home/student/index.html` doesn't exist on either of your worker nodes currently.

18. Use `kubectl get pods`

Example output:

```bash

```

<br/>

19. And then use `kubectl describe pod` to see a message about what's going on.

<details><summary>show command</summary>
<p>

```bash
NAME                       READY   STATUS              RESTARTS   AGE
lab3web-84954c7d89-hhpfd   0/1     ContainerCreating   0          25s
lab3web-84954c7d89-lwblq   0/1     ContainerCreating   0          25s
lab3web-84954c7d89-mpmb5   0/1     ContainerCreating   0          25s
```

</p>
</details>
<br/>

20. Let's use `kubectl describe pod` to further investigate.

<details><summary>show command</summary>
<p>

```bash
kubectl describe pod lab3web-84954c7d89-mpmb5
```

</p>
</details>
<br/>

Example output:

```bash
...
Events:
  Type     Reason       Age                From               Message
  ----     ------       ----               ----               -------
  Normal   Scheduled    49s                default-scheduler  Successfully assigned default/lab3web-84954c7d89-hhpfd to k8s-worker-0
  Warning  FailedMount  17s (x7 over 49s)  kubelet            MountVolume.SetUp failed for volume "homedir" : hostPath type check failed: /home/student/index.html is not a file
```

<br/>

21. The file doesn't exist on either of the worker nodes!

22. Copy the file `~/index.html` to both of the worker nodes. Use the **scp** command with a source of `~/index.html` and a destination of `student@k8s-worker-0:~/index.html` (and then again to `k8s-worker-1`). You will need to provide your password, which is `P@$$w0rd`, both times.

<details><summary>show command</summary>
<p>

```bash
scp ~/index.html student@k8s-worker-0:~/index.html
scp ~/index.html student@k8s-worker-1:~/index.html
```

</p>
</details>
<br/>

23. **cURL** the service again. This time you should see your customised home page. This seems like a painful and non-scalable approach to solve this particular problem, does't it? This is where `PersistentVolumes`, shared amongst all of the pods in the cluster, might come in handy. We're going to solve it a different way, using `ConfigMaps`.

24. Delete the deployment, but not the service.

<details><summary>show command</summary>
<p>

```bash
kubectl delete deployment lab3web
```

</p>
</details>
<br/>

## 1.2 ConfigMaps

### Task 3 - create a ConfigMap from your index.html file

25. Edit your index.html file to change the welcome message. To make it clearer what's going on, think about adding the term "configmap" to the message.

```html
<html><body><h1>Welcome to my home page ConfigMap!</h1></body></html>
```

26. Create a ConfigMap from the updated `index.html` file.

<details><summary>show command</summary>
<p>

```bash
kubectl create configmap homepage --from-file ~/index.html
```

</p>
</details>
<br/>

27. Edit (a copy of) the lab3web.yaml file to replace the `hostPath` volume with a `configMap` volume, using the newly-created `homepage` configmap. You also need to change the `path` of the volume mount to only include the `htdocs` directory. [NOTE: We could now add additional files to the configmap and they'd be mounted in the same directory]

<details><summary>show command</summary>
<p>

```yaml
        volumeMounts:
        - name: homepage
          mountPath: /usr/local/apache2/htdocs
      volumes:
      - name: homepage
        configMap:
          name: homepage
```

</p>
</details>
<br/>

28. Delete and then recreate the `lab3web` deployment.

<details><summary>show command</summary>
<p>

```bash
kubectl delete deployment lab3web
kubectl create deploy lab3webconfigmap.yaml # whatever your edited file is called
```

</p>
</details>
<br/>

29. And **cURL** your lab3web service IP address again.

<details><summary>show command</summary>
<p>

```bash
 curl 10.101.251.165
```

</p>
</details>
<br/>

Example output:

```bash
<html><body><h1>Welcome to my home page ConfigMap!</h1></body></html>
```

<br/>

### Task 4 - modify the simple frontend deployment to use configmaps in environment variables

The simple frontend application also has a placeholder for a `COLOUR` environment variable. We're going to add different values for that in our different namespaces. 

30. Create a ConfigMap in both the development and production namespaces. Name the configmap `settings` and create a `colour` setting from a literal value with different values in each namespace. Purple and green are good choices, but feel free to use other values.

<details><summary>show command</summary>
<p>

```bash
kubectl create configmap settings --from-literal=colour=purple --namespace development
kubectl create configmap settings --from-literal=colour=green --namespace production
```

</p>
</details>
<br/>

31. Edit (a copy of) the lab2frontend.yaml file to add another `env` setting named `COLOUR` that gets its value from a `configMapKeyRef` with a `name` of `settings` and a `key` of `colour`. Maybe also find and replace all the `lab2frontend`s with `lab3frontend`s.

<details><summary>show YAML</summary>
<p>

```yaml
        env:
        - name: COLOUR
          valueFrom:
            configMapKeyRef:
              name: settings
              key: colour        
```

</p>
</details>
<br/>

32. Create a sfe deployment in both the production and development namespaces. You did this in the second lab.

<details><summary>show commands</summary>
<p>

```bash
kubectl apply -f lab3frontend.yaml --namespace development
kubectl apply -f lab3frontend.yaml -n production
```

</p>
</details>
<br/>

33. Create a NodePort service exposing the deployment in both namespaces. Remember that the application is running on port 8080.

<details><summary>show command</summary>
<p>

```bash
kubectl expose deployment lab3frontend --port 8080 --type NodePort --namespace production 
kubectl expose deployment lab3frontend --port 8080 --type NodePort -n development
```

</p>
</details>
<br/>

34. Obtain the nodeport of both services and then browse to them.

<details><summary>show command</summary>
<p>

```bash
kubectl get service lab3frontend --all-namespaces
```

</p>
</details>
<br/>

<details><summary>Stretch goal - optional exercise</summary>
<p>

35. **OPTIONAL Stretch goal** also create the configmap in the test namespace and create and expose the frontend deployment therein.

</p>
</details>
<br/>

## 1.3 Secrets

### Task 5 - work with secrets

36. Create a `secret` called `secrets` from a literal value with a key of `password` in the dev and prod namespaces, with different values for each password.

<details><summary>show command</summary>
<p>

```bash
kubectl create secret generic secrets --from-literal password=MySecretPhrase --namespace development
kubectl create secret generic secrets --from-literal password=ReallySecret --namespace production
```

</p>
</details>
<br/>

37. Edit (a copy of) the lab3frontend.yaml file to add a `volume` to the deployment with a `name` of `secret-volume` and a `type` of `secret`, referencing your newly-created `secret`. Add a `volumeMount` to the container that mounts your secret at `/data`

<details><summary>show YAML</summary>
<p>

```yaml
...
        volumeMounts:
        - name: secret-volume
          mountPath: /data
      volumes:
      - name: secret-volume
        secret:
          secretName: secrets
```

</p>
</details>
<br/>

38. Obtain the nodeport of both services and then browse to them.

<details><summary>show command</summary>
<p>

```bash
kubectl get service lab3frontend --all-namespaces
```

</p>
</details>
<br/>

<details><summary>Stretch goal - optional exercise</summary>
<p>

39. **Optional stretch goal** add the secret to the `test` namespace as well, then create and expose the frontend deployment therein.

</p>
</details>
<br/>

30. Tidy up. Delete all three deployments and the three services.

<details><summary>show command</summary>
<p>

```bash
kubectl delete deploy lab3frontend -n production
kubectl delete deploy lab3frontend -n development
kubectl delete service lab3frontend -n production
kubectl delete service lab3frontend -n development
kubectl delete deploy lab3web
kubectl delete service lab3web
```

</p>
</details>
<br/>

31. That's it, you're done! Let your instructor know that you've finished the lab.