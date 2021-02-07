# Services

# Create a service

You can use kubect expose to do so imperatively:

```
k expose deploy/web --port 80
```

This creates a service:

```
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
web          ClusterIP   10.101.239.251   <none>        80/TCP    14s
```

# Service manifest

```
apiVersion: v1
kind: Service
metadata:
  labels:
    app: web
  name: web
spec:
  type: ClusterIP
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: web
```

Types:
- ClusterIP
- NodePort
- LoadBalancer
- ExternalName

## Port forward to service

```
k port-forward svc/web 8080:80
```

Above, port 8080 on localhost is forwarded to the port 80 of the service.

```
curl http://localhost:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
... 
```

## Endpoints

Run **k describe svc web**:

```
Name:              web
Namespace:         default
Labels:            app=web
Annotations:       <none>
Selector:          app=web
Type:              ClusterIP
IP:                10.100.118.163
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.244.0.21:80,10.244.0.22:80,10.244.0.23:80
Session Affinity:  None
Events:            <none>
```

Above, the service knows of three endpoints. You can list the endpoints as well:

```
k get ep
```

Result:

```
NAME         ENDPOINTS                                      AGE
web          10.244.0.21:80,10.244.0.22:80,10.244.0.23:80   4m15s
```
