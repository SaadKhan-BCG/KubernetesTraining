# Kubernetes Training
The examples included here are complete and runnable, including on Mac with docker desktop and kubernetes mode enabled.
They are intended to act as references for how to migrate an app to kubernetes.

## Ex 0 (Not really an exercise)
Run the docker example:
```
cd docker
docker compose up
```

Open up http://localhost (Note https doesn't work) and click around 
Once you are done, close the app down (a simple ctrl-c, to exit out)

## Ex 1
Open the new k8s directory and take a look at the files there (Note they are not populated at all)
Using what we have learned, see if you can decide what resource types should fit in to the files.
You can refer to the kubernetes docs https://kubernetes.io/docs/concepts/workloads/controllers/ for different resource types.

### Bonus Task: 
Once you have identified the resource type open the docs for that type e.g. 
if you decide "Pod" (Hint its not Pod!) search the docs for the Pod resource type. You will find an example template file there (If you've picked right it will be an nginx instance and can be ran as is).
See if you can copy-paste it, run it and have a thing about what changes we would make for the wordpress app.

You can run a deployment with the command:
```
kubectl apply -f <file_name>
```

If you wish to check it deployed you can run:
```
kubectl get pods -> gets you a list of running pods
kubectl logs  <pod_name> -> get the pod name from the prior line 
```