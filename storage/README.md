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

# Azure Storage

Volumes:
- Azure Disk: Azure Premium or Standard; mounted as ReadWriteOnce which means only available to a single pod
- Azure Files: SMB 3.0 storage backed by Azure Standard or Premium storage; ReadWriteMany for access by multiple pods

Note that the above are just Kubernetes volume types, next to other types such as emptyDir, secret, configMap, ...

Persistent volumes:
- volumes created as part of the pod only exist until the pod = deleted
- PV (persistent volume) can persist beyond the lifetime of a pod
- use Azure Disk or Files
- can be created **statically** by the admin OR
- can be created **dynamically** by Kubernetes (uses a StorageClass)

Storage classes:
- default: Azure Standard SSD to create a managed disk
- managed-premium
- azurefile
- azurefile-premium

With CSI:
- managed-csi: Azure Standard SSD
- managed-csi-premium
- azurefile-csi
- azurefils-csi-premium

**Note:** by default, the store classes remove the storage if the PV is deleted; you can create a StorageClass that **retains** the volume when the PV is deleted:

**Note:** from Kubernetes version 1.21, CSI drivers will be used only; for now using the CSI drivers instead of storage support in core Kubernetes is in preview; see https://docs.microsoft.com/en-us/azure/aks/csi-storage-drivers

```
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: managed-premium-retain
provisioner: kubernetes.io/azure-disk
reclaimPolicy: Retain
parameters:
  storageaccounttype: Premium_LRS
  kind: Managed
```

PVC (persisten volume claim) can create a disk or file storage of a StorageClass, the requested access mode and size.

Example:

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: azure-managed-disk
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: managed-premium
  resources:
    requests:
      storage: 5Gi

```

The pod can use it as follows:

```
kind: Pod
apiVersion: v1
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx
      volumeMounts:
      - mountPath: "/mnt/azure"
        name: volume
  volumes:
    - name: volume
      persistentVolumeClaim:
        claimName: azure-managed-disk
```

## Azure Static Disk

See file azure-static-disk.yaml

You need to first create the disk:

```
az disk create \
  --resource-group MC_myResourceGroup_myAKSCluster_eastus \
  --name myAKSDisk \
  --size-gb 20 \
  --query id --output tsv
```

The command displays the resource ID. You need to provide that in the YAML.

## Azure Static File

See file azure-static-files.yaml

You need to create the file share first:

```
# Change these four parameters as needed for your own environment
AKS_PERS_STORAGE_ACCOUNT_NAME=mystorageaccount$RANDOM
AKS_PERS_RESOURCE_GROUP=myAKSShare
AKS_PERS_LOCATION=eastus
AKS_PERS_SHARE_NAME=aksshare

# Create a resource group
az group create --name $AKS_PERS_RESOURCE_GROUP --location $AKS_PERS_LOCATION

# Create a storage account
az storage account create -n $AKS_PERS_STORAGE_ACCOUNT_NAME -g $AKS_PERS_RESOURCE_GROUP -l $AKS_PERS_LOCATION --sku Standard_LRS

# Export the connection string as an environment variable, this is used when creating the Azure file share
export AZURE_STORAGE_CONNECTION_STRING=$(az storage account show-connection-string -n $AKS_PERS_STORAGE_ACCOUNT_NAME -g $AKS_PERS_RESOURCE_GROUP -o tsv)

# Create the file share
az storage share create -n $AKS_PERS_SHARE_NAME --connection-string $AZURE_STORAGE_CONNECTION_STRING

# Get storage account key
STORAGE_KEY=$(az storage account keys list --resource-group $AKS_PERS_RESOURCE_GROUP --account-name $AKS_PERS_STORAGE_ACCOUNT_NAME --query "[0].value" -o tsv)

# Echo storage account name and key
echo Storage account name: $AKS_PERS_STORAGE_ACCOUNT_NAME
echo Storage account key: $STORAGE_KEY
```

The storage account name and key is required for a Kubernetes secret:

```
kubectl create secret generic azure-secret --from-literal=azurestorageaccountname=$AKS_PERS_STORAGE_ACCOUNT_NAME --from-literal=azurestorageaccountkey=$STORAGE_KEY
```

Provide a reference to the storage account and secret in the YAML:

```
volumes:
  - name: azure
    azureFile:
      # secret contains storage account key
      secretName: azure-secret
      # share needs to be created in advance
      shareName: aksshare
      readOnly: false
```
