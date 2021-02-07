# Secrets

## Create with kubectl

There are three types of secrets:
- docker-registry
- generic
- tls

Let's create a generic secret:

```
k create secret generic mysecret --from-literal=password=password
```

Describe the secret:

```
k describe secret mysecret
```

This only states: secret password = 8 bytes

```
k get secret mysecret -oyaml
```

Now you see:

```
data:
  password: cGFzc3dvcmQ=
```

The password is base64 encoded:

```
echo cGFzc3dvcmQ= | base64 -d
```

## From a manifest

```
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
data:
  password: cGFzc3dvcmQ=
```

When you create a secret with manifest, you are responsible for encoding it.