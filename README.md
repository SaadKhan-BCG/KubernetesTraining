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
 