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


# Ex 2
Populate the TODOs pieces in the app-deployment and db-deployment. 

**Challenge:** Try and do it without reading the Hints below

**Hint 1.** 

Take a look at the nginx example here https://kubernetes.io/docs/concepts/workloads/controllers/deployment/ 
see what values they have? can we copy these? if not how do they change?

**Hint 2.**

The selector in the deployment needs to align with the values for the Service so they point to each other.
If you look at the example as recommended in hint 1, youll see:
```
matchLabels:
  app: nginx
```
Under the selector. What labels have we given our Service? is it also app:nginx? or is it something else

### Now lets test it out!

### Run Commands
First Create the namespace:

```
kubectl create namespace wordpress 
```

And then the secret to store db password
```
kubectl -n wordpress create secret generic mysql-pass \
    --from-literal=password='password'
```

And finally 
```
kubectl apply -f .
```

## Bonus Task:
Change the name of the db-deployment and redeploy. What happens?
See if you can correct it.