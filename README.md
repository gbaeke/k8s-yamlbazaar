# k8s-yamlbazaar

YAML examples to get started

## Useful general commands

Get client and server version:

```
k version
```

Get cluster information

```
k cluster-info
```

List the api versions (kind, 1.19.1). E.g., shows networking.k8s.io/v1 and /v1beta1

```
k api-versions
```

List api resources. Gives name, short nale, api group, namespaced or not and kind (e.g.pods, po, empty api group = core api, true, Pod)

```
k api-resources
```

## Kube config

View your configuration:

```
k config view
```

Explanation of the result (in my case):
- two clusters
    - Azure cluster (https://xyz.hcp.westeurope.azmk8s.io:443)
    - kind (https://127.0.0.1:45019)
- two contexts (context links cluster to user)
    - Azure cluster --> clusterUser_RGNAME_CLUSTERNAME
    - kind --> kind-kind
- two users
    - Azure cluster --> clusterUser_RGNAME_CLUSTERNAME
        - Azure cluster: AAD integrated (token information)
        - kind --> client-certificate-data (REDACTED) and client-key-data (REDACTED)

If you do not want redacted information:

```
k config view --raw
```

You will see the certificate and key data as base64 encoded strings. To check the certificate:

```
kubectl config view --minify --raw --output 'jsonpath={..client-certificate-data}' | base64 -d | openssl x509 -text -out - | head
```

This will show something like:

```
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 8776316788637609435 (0x79cbbd9ecf0aaddb)
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN = kubernetes
        Validity
            Not Before: Feb  7 16:59:32 2021 GMT
            Not After : Feb  7 16:59:34 2022 GMT
        Subject: O = system:masters, CN = kubernetes-admin
```

The user here is **kubernetes-admin**. The organization (O) is used as the group. 

The group is used in a ClusterRoleBinding:

```
k get clusterrolebindings cluster-admin -o yaml
```

In the output of the above command, you will find the subjects: section with a reference to a Group called system:masters.

