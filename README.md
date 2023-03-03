# Kubernetes Training
The examples included here are complete and runnable, including on Mac with docker desktop and kubernetes mode enabled.
They are intended to act as references for how to migrate an app to kubernetes.

## Docker

## Run Commands
A simple ``docker compose up`` will start up the app on localhost

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
kubectl apply -f .
```

## K8s Dashboard
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


### Cleanup
Delete deployment simply by replacing "apply" with "delete" e.g. (to delete wordpress deployment)
```kubectl delete -f .```

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
 
# Ex 7.
Cluster Roles are a great way to limit access to your cluster and can be super fine-grained.
By default we are all currently using a special user in kubernetes called system:master, this user has total admin access.
But what about if we want to give another team just read only access for debugging?

Let's create a Read Only User.
One small disclaimer, we will be creating a "role" and a "service account". we will then test this user by assigning it to a pod.
This process has 3 parts:
- A cluster role: The permission definition
- Service account: the entity you wish to assign permissions to
- ClusterRoleBinding: A mapping between cluster roles and service accounts

You may then ask how do we actually use this? For this training, we will be assigning our service account to a pod, which we can log in to and check it's permissions.
In practise, we often want to assign our role to a physical person, this process unfortunately is very dependent on where your kubernetes cluster is.

For example in AWS, once you have created your role and service account, you edit something called the "aws-auth-configmap" which is a mapping from that kubernetes service account to an AWS IAM Role (which a human can then use).
Similarly all other cloud vendors have their own process for assigning the service account to.

Therefore, we will go as far as creating our service account with permissions, in your cases you will need to refer to the official documentation for your provider to find out whats next.

Open up the file helm-read-only-cluster-role.yml, you will see a few TODO comments

1. Decide on the permissions you want to assign under "Verbs"
   Note the available options are:  
```
["get", "list", "watch", "create", "update", "patch", "delete"]
``` 
  What does our read only user need?
  Note here you need to replicate it 3 times, once for each type of kubernetes resource.
  If youre interested in understanding more about the fine-grained roles you can read about it here https://kubernetes.io/docs/reference/access-authn-authz/rbac/  
2. In the ClusterRoleBinding finish the mapping between Service Account and ClusterRole
    Hint. for your convenience, the service account, clusterrole and binding are all in the same file, so to finish the binding you only need to populate the name of the serviceaccount and clusterrole with their corresponding names in that file

Apply this change as usual with the helm upgrade command and lets test our role
Open K9s and switch to the default namespace :namespace -> default
scroll down to the new pod that should be created, called test
type ``s`` to open a shell inside that pod (Similar to a docker exec)

This pod is special, it has kubectl installed and has been granted our new Read only service account
if you have set your permissions right you should be able to run 
```

kubectl get pods -n wordpress
```

picking a running instance you should NOT be able to do a 
```
kubectl kill pod <pod_id>
```

Try some other commands now. See what works and what doesn't.

once you have finished inside it, exit the pod and please delete it using k9s (ctrl-d). If you wish to make changes and test again you can always re-run our helm upgrade command

### Bonus:
Try and change our read only role to only have read only access to Pods, not deployments. (You can rely on the docs listed about).
Hint here deployments are part of ``apiVersion: apps/v1``, whereas pods are part of ``apiVersion: v1`` (This is where those 3 different permission blocks we set matter)

# Ex 9. 
Adding Liveness and Readiness Probes.
This is a very similar concept to docker, liveness/readiness probes give us the confidence our app is actually up and running
If you have a custom healthcheck in your app its common to use this, for our example we're keeping it simple:

Add the following block under the volumeMount in the helm/templates/app-deployment.yml:
For reference you can see the docs https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#http-probes

Note be careful with indentation, the livenessProbe and readinessProbe keys should be inline with the volumeMounts key.

```yaml
livenessProbe: 
  initialDelaySeconds: 120
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 6
  successThreshold: 1
  httpGet: null
  tcpSocket:
    port: 80
readinessProbe:
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 6
  successThreshold: 1
  httpGet: null
  tcpSocket:
    port: 80
```

Again as usual, upgrade our cluster.

**Note** when looking for success/failure of liveness probes its import to check the "describe" not the logs

e.g in K9s, pick your deployment and press ``d``, then scroll to the bottom it will show the successes/failures

### Bonus
Let's play around with this a little, try tweaking the httpGet (This is the endpoint our healthcheck is hitting)
Have a look through the docs and try messing around with the other paramters.