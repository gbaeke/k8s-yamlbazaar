# Storage

# emptyDir

emptyDir volumes are backed by storage on the nodes. Good for scratch space or tests like this! 😀

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
      volumeMounts:
        - mountPath: /mydata
          name: data
    - name: other
      image: busybox
      command:
        - sleep
        - "3600"
      volumeMounts:
        - mountPath: /mydata
          name: data
  volumes:
    - name: data
      emptyDir: {}
        
```

You can access the volume from both containers in the pod. The containers can use different mount points.
