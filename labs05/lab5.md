# Lab 5 - DaemonSets, Jobs and Helm

## 5.1 DaemonSets

Look to see if there are any ds running on the cluster. Look in all namespaces. Use --output=wide.

Create a ds using the httpd image. It's very similar to a deployment, but has a different kind and no replicas property.

Expose it and browse to it

(stretch: note that you only have two pods running whereas the system ds have 3. Can you work out why that is (and make it so yours works the same way) hint: try describing the pods and describing the nodes)

## 5.2 Jobs

Create a job that prints a random number

job.yaml:
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: randoms
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: example
        image: python
        command:
        - python3
        - -c
        - |
          import random
          print(random.randrange(1,100))
```

Get the pod's logs

Change it to have 3 completions

apply

Get all 3 pods' logs

Change it to 10 completions, parallelism 3.

Immediately start watching pods

Get all 10 pods' logs

Must be an easy way, right?

for pod in $(kubectl get pods -l=job-name=randoms -o name); do kubectl logs $pod; done

## 5.3 CronJobs

Turn the Job into a CronJob that runs once a minute.

Take a 10-minute break.

Check logs.

Delete CronJob

## 5.4 Helm

Install helm

search hub

add bitnami repo

install apache "my-web"

upgrade, changing the service type to clusterIP.

(stretch: Create an ingress rule for it at web.yourip.nip.io)

