# Pods

## Creating a pod

See https://kubernetes.io/docs/concepts/workloads/pods/

Create pod.yaml:

```
apiVersion: v1
kind: Pod
metadata:
  name: web
  labels:
    app: web
spec:
  containers:
    - name: web
      image: nginx
      ports:
        - name: web
          containerPort: 80
          protocol: TCP
```

Create the pod:

```
k create -f pod.yaml
```

What can you use in the pod's spec?

```
k explain pod.spec
```

For example, to disable the autmatic mounting of the service account token:

```
spec:
  automountServiceAccountToken: false
```

Above we create a static pod. It is managed by the kubelet on the node on which it is scheduled. The kubelet can restart the pod but the pod will not be scheduled on another node (e.g., after node failure).

## Multiple containers

```
apiVersion: v1
kind: Pod
metadata:
  name: web
  labels:
    app: web
spec:
  containers:
    - name: web
      image: nginx
      ports:
        - name: web
          containerPort: 80
          protocol: TCP
    - name: other
      image: busybox
      command:
        - sleep
        - "3600"
```

Result of **k get po**:

```
NAME   READY   STATUS    RESTARTS   AGE
web    2/2     Running   0          62s
```

Get shell to container:

```
k exec -it web -- sh
```

Uses the first container, web. Use -c to specify the container to use:

```
k exec -it web -c other -- sh
```

## initContainers

```
apiVersion: v1
kind: Pod
metadata:
  name: web
  labels:
    app: web
spec:
  containers:
    - name: web
      image: nginx
      ports:
        - name: web
          containerPort: 80
          protocol: TCP
  initContainers:
    - name: init1
      image: busybox
      command:
        - sleep
        - "10"
    - name: init2
      image: busybox
      command:
        - sleep
        - "10"
```

The init containers will need to finish running before the web container can run. The init containers needs to run successfully. Multiple init containers are run in sequence in the order they appear.

See https://kubernetes.io/docs/concepts/workloads/pods/init-containers/

