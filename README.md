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

## Now lets test it out!

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

# Ex 3.

Run the application now using the run commands from ex3. Did your example work before?

Using the k9s tool take a look at your running app

```
k9s -> Starts k9s in a terminal (usually defaults to default namespace
:namespace -> shows list of namespaces
use the arrow keys to navigate to wordpress and press enter
Note. from now on k9s has assigned it a shortcut number, you can see this at the top

```

Pick on a pod and press l to see the logs, does it look ok?


## Bonus
Explore the other functionality of k9s
Switch from Pods to the "Deployment" view ``:deployment``, and then select our app deployment and press ``e``
Use the vim editor it gives you to change something in the deployment, e.g. increase the replicas.
Close it and see what happens.

# Ex 4.
### Run the K8s Dashboard
Start kubernetes dashboard with the following: 
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.6.1/aio/deploy/recommended.yaml
```

Generate token with:
```
kubectl apply -f dashboard-account.yml
kubectl -n kubernetes-dashboard create token admin-user
```

Access UI:
- Generate port-forward:
  - K9s Commands:
    - :svc -> (switch to view services)
    - (Navigate to service kubernetes-dashboard) <shift-f>
    - Accept (and not the "local port" chose, usually 8443)
    - Navigate to locahost:<local_port> e.g. localhost:8443
    - Login with token generated above (You may need to click person icon in top right and click sign in first)

  
### Bonus
Open up k9s side by side, and compare. What do you like about one vs the other? What limitations do each tool have


### Cleanup
Delete deployment simply by replacing "apply" with "delete" e.g. (to delete wordpress deployment)
```kubectl delete -f .```

# Ex 5.
Introduction to Helm!
Open up the new helm directory and take a look at the Chart.yaml, values.yaml files.
Add a dependency mysql to the chart.yaml and deploy it, look what happens.

Copy the following block under the "dependencies tab"
```yaml
dependencies:
  - name: mysql
    version: 9.4.1
    repository: https://charts.bitnami.com/bitnami
```

Assuming namespace was created as per previous, just run: 
```
helm -n wordpress install wordpress helm/ --values helm/values.yaml
```

If you wish to then edit a running chart replace install with upgrade: 
```
helm -n wordpress upgrade wordpress helm/ --values helm/values.yaml
```
 


## Bonus task:
Using the docs for the mysql chart https://artifacthub.io/packages/helm/bitnami/mysql
Read through the parameters and try and apply our wordpress label to all the chart's resources.
Use the upgrade command above and then investigate in k9s to verify your label was assigned


# Ex 6.
Let's migrate our kubernetes manifests to helm.

Open helm/templates/app-deployment.yaml -> You will see I have copy pasted the exact file from k8s/app-deployment.yml
Try running it? What Happens?

Remember: to apply changes we use
```
helm -n wordpress upgrade wordpress helm/ --values helm/values.yaml
```

If you check the logs (remember we have k9s) you will see it fail to connect to the DB.
This is because the new helm chart we installed for mysql generates its own secret for the password, we need to now use this.
Update the environment variable in the app-deployment to the following: (note the name and key have changed)

```yaml
- name: WORDPRESS_DB_PASSWORD
  valueFrom:
    secretKeyRef:
      name: wordpress-mysql
      key: mysql-root-password
```

## Templates
One of the big advantages of helm is the ability to template variables for easy re-use. Lets try this out

under the environment variable "DB_HOST" try replacing the hard coded value
``value: wordpress-mysql`` with ``value: {{ .Values.app.dbHost }}``

Now open the values.yaml and look there for dbHost you can see it here!
Note helm supports the use of multiple values.yaml files so you can create a baseline, and a different file for different environments.
lets try this!

duplicate the values.yaml, creating a second values.dev.yaml
now remove the ``dbHost: wordpress-mysql`` from the original

try and re-run 
```
helm -n wordpress upgrade wordpress helm/ --values helm/values.yaml
```

What happens?
We havent told helm to use our second values.yaml file, lets fix this (Note values.yaml files are read left to right and overwritten if you have the same key)
```
helm -n wordpress upgrade wordpress helm/ --values helm/values.yaml --values helm/values.dev.yaml
```


### Bonus tasks:

**DB Secret Value**
Have a look at the docs again for the db (https://artifacthub.io/packages/helm/bitnami/mysql)
There is a parameter there to tell the chart to use an existing db secret instead of its own. Can you find it?
Try and use this parameter to point at our old db secret instead.

Note this will be really helpful in practise, where we often want to set the password ourselves e.g. storing it in AWS Secrets Manager

You can test it works by reverting the helm/templates/app-deployment.yml to match the k8s/app-deployment.yml and check it still works.

**Templates**
Have another read through of the app-deployment and think what else you would want to template and control, e.g. the replicas
Try and add more templated variables, testing them out.