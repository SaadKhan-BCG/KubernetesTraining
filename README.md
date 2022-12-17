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
kubectl appply -f .
```

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
 