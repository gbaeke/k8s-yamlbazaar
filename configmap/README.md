# ConfigMap

# Creating a ConfigMap with kubectl

ConfigMaps store key/value pairs. You can use literals as below:

```
k create cm config --from-literal=mykey=myvalue
```

To use a file, for instance, myfile.txt. First create a simple file:

```
cat <<EOF > myfile.txt
 No idea what to write here...
EOF
```

```
 k create cm config-file --from-file=myfile=./myfile.txt
```

Let's look at the first ConfigMap:

```
k describe cm config
```

The data shows mykey equals myvalue:

```
Name:         config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
mykey:
----
myvalue
Events:  <none>
```

For the second ConfigMap:

```
Name:         config-file
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
myfile:
----
 No idea what to write here...

Events:  <none>
```

In YAML:

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: config
  namespace: default
data:
  mykey: myvalue
```

The file:

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-file
  namespace: default
data:
  myfile: |2
     No idea what to write here...
```

Of course, a ConfigMap can have multiple key/value pairs.

## Manifest

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: config
  namespace: default
data:
  mykey: myvalue
  otherkey: othervalue
  myfile: |
    This is a file!
    There are two lines
```

## Usage in pod

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
        - mountPath: /myconfig
          name: config
      env:
        - name: MYKEY
          valueFrom:
            configMapKeyRef:
              key: mykey
              name: config
  volumes:
    - name: config
      configMap:
        name: config
        
```

