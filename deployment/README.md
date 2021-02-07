# Deployments

## Creating a deployment

Create with kubectl:

```
k create deployment --image nginx web
```

Result: 

```
NAME                      READY   STATUS    RESTARTS   AGE
pod/web-96d5df5c8-l2lx2   1/1     Running   0          17s

NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/web   1/1     1            1           17s

NAME                            DESIRED   CURRENT   READY   AGE
replicaset.apps/web-96d5df5c8   1         1         1       17s
```

Resulting objects:
- deployment: web
- replicaset: web-rsid
- pod: web-rsid-id

Generate deployment manifest with --dry-run:

```
 k create deployment --image nginx web --dry-run=client -oyaml
```

Modified the above result to:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: web
  name: web
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web
  strategy:
    rollingUpdate:
      maxSurge: 25% # default
      maxUnavailable: 25% # default
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - image: nginx:1.18  # use older image at time of writing
        name: nginx
```

## Scale deployment

```
k scale deploy web --replicas=3
```

## Set the image

```
k set image deploy/web nginx=nginx:1.19
```

This triggers a rollingUpdate (selected strategy). There are now two replica sets:

```
NAME             DESIRED   CURRENT   READY   AGE
web-6c57bdf5f4   0         0         0       3m53s
web-bc7cc9f65    3         3         3       84s
```

Check rollout status:

```
k rollout status deploy/web
```

Check history:

```
k rollout history deploy/web
```

You will see two revisions.

Undo previous rollout:

```
k rollout undo deploy/web
```

Previous replica set will now have running pods. The new replica set does not. The image will be 1.18 again.

