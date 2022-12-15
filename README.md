# Kubernetes Training
The examples included here are complete and runnable, including on Mac with docker desktop and kubernetes mode enabled.
They are intended to act as references for how to migrate an app to kubernetes.

## Docker

## Run Commands
A simple ``docker compose up`` will start up the app on localhost

### Instructor's Notes
This is the starting point of the exercise, introduce the 2 services, app and db, and their volumes.
Highlight the use of env vars to pass info including the password (bad practise)


## Kubernetes

## Run Commands
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
kubectl appply -f .
```

### Instructor's Notes
Please note from beginning this course is designed to be ran on docker-desktop kubernetes instead of a live cluster. Desktop k8s is not bad, but has its flaws and hence there will be isssues in this course i may raise where "this doesnt work locallt" 

Starting from the Docker example, first Identify what components we could need:
- App
  - Service - To handle networking to the k8s instance
  - Deployment - Deployment type is most appropriate for running the app
  - Volume - To store the data, in this case we can use a PersistentVolumeClaim to ensure data isn't lost
  - Config:
    - (Optional) we could move a bunch of the environment variables to a configmap, or just pass directly since its not many and not shared to anyone else
    - Secret To store the DB password
- DB
  - Service
  - Deployment - In theory a statefulset would be better to share the load, but for this simple example we will just use a deployment of 1 instance for simplicity
    - Will circle back to this in the helm example where we let someone else build it for us
  - Volume
  - Secret

Once these pieces are identified we can start building them out, rely on official docs eg https://kubernetes.io/docs/concepts/workloads/controllers/deployment/ to provide syntax to copy-paste


## Helm

## Run Commands

Assuming namespace was created as per previous, just run: 
```
helm -n wordpress install wordpress helm/ --values helm/values.yaml
```

or to make changes 
```
helm -n wordpress upgrade wordpress helm/ --values helm/values.yaml
```

### Instructor's Notes
Why Helm?
Helm is a packaging tool for k8s, enabling you to easily build and combine k8s apps
Immediate value for us, is it allows us to rely on pre-built k8s apps instead of re-implelementing ourselves (remeber the DB? we can rely on a helm chart for this)
Artifact hub can be used to search for helm charts, in particular rely on the official kubernetes ones and bitnami (who produce a lot of useful ones)
e.g. https://artifacthub.io/packages/helm/bitnami/mysql

Other main advantage of helm is easy templating. This allows us to reuse k8s apps and redeploy with different config e.g. for dev/staging/prod & separates config from code better

to start show how we can generate a helm chart with ``helm create wordpress-example``
This is good practise and populates a lot of useful pieces, but to make this training a little quicker we wont touch everything it creates and will start from scratch ourselves

First create a Chart.yml and values.yml
Add dependency for mysql bitnami chart, add commonLabels to it via the values.yml and deploy
Then add in the wordpress app via templates
Start templating out variables

#### Autoscaling
Autoscaling is easier to introduce here as it requires the metrics-server, simply install this via helm and deploy (note the awkward tweaking to make it work locally)
Then add a HorizontalPodAutoscaler resource (refer to docs for how) and thats it.
Quick note on this, autoscaling doesn't work on docker desktop as the metrics-server doesnt pull metrics correctly records everything as 0

### RBAC and roles
By default we are working with the k8s admin user here, this is fine for our superusers but for everyone else not so good.
E.g. if you have 2 client apps on your cluster, its often a good idea to separate them by namespace, => you wont want to give client team A access to client B's deployment
Also admin can do many dangerous things damaging cluster, v few should use it you want to give out more granualar roles e.g. read-only, Access to pods/deployments/statefulsets but not secrets etc
Also similarly you may want to delegate certain permissions to the app itself, e.g. you may want your api to have jobs/create access so it can fire off jobs to do compute

K8s can handle this through a v detailed role management, this example will cover a simple read-only case

Create a ClusterRole with permissions we need (note cluster roles are for every namespace, there is also a Role which is for the given namespace, and often better to hand out)
Note similar to AWS policy, a k8s role by itself is useless, it needs to be assigned to someone

Create a Service Account

Then create a clusterrolebinding (note rolebindings also exist for namespaces)

This grants the given service account the specified access
Test this using the read-only-test.yml (dont bother creating this just copy-paste from solution, exec in to it and demonstrate it has kubectl get pods access)

How to use it now?
Service Accounts can be used to delegate access level to k8s services, but what about me? i access using a kubeconfig (show mine) which is assigned to the system:master clusterrole a default one.
You can create more of these but it depends on your cluster e.g. in AWS with eks they have something called the aws-auth configmap which is used to map a k8s serviceaccount to an AWS role/user, you should read the docs of AWS/Azure/GCP etc on how to do this.
For a self managed cluster this is a pain, there are guides, i wont cover as its very unlikely any of us will have to do this 