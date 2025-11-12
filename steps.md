Step 1: Create Namespace
    
> kubectl create namespace ns-eks-course
  
Step 2: Configure Storage Class
Create a file named gp2-storage-class.yaml:
```
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: gp2
provisioner: ebs.csi.aws.com
parameters:
  type: gp2
reclaimPolicy: Retain
mountOptions:
  - debug
```
       
Apply the storage class:    
> kubectl -f gp2-storage-class.yaml replace --force --namespace=ns-eks-course

Step 3: Set Up IAM OIDC Provider
```    
eksctl utils associate-iam-oidc-provider \
    --region us-west-2 \
    --cluster EKS-course-cluster \
    --approve
```
        
Step 4: Create Service Account
```    
eksctl create iamserviceaccount \
    --name ebs-csi-controller-sa \
    --namespace kube-system \
    --cluster EKS-course-cluster \
    --attach-policy-arn arn:aws:iam::3XXXXXXXXXX5:policy/AmazonEKS_EBS_CSI_Driver_Policy \
    --approve \
    --region us-west-2
````
      
Step 5: Install EBS CSI Driver
    
>> kubectl apply -k "github.com/kubernetes-sigs/aws-ebs-csi-driver/deploy/kubernetes/overlays/stable/?ref=release-1.21"
  
Step 6: Verify Installation

- Check controller pod status:

> kubectl get pods -n kube-system | grep ebs-csi-controller
    
- Check node pods status:

> kubectl get pods -n kube-system | grep ebs-csi-node

## Troubleshooting
- Common Issue: PVC Binding Failure

If you encounter the following error:
```    
Events:
  Type    Reason         Age               From                         Message
  ----    ------         ----              ----                         -------
  Normal  FailedBinding  5s (x7 over 85s)  persistentvolume-controller  no persistent volumes available for this claim and no storage class is set
```

## Solution: Configure PVC with Storage Class
- Example PVC configuration:
```    
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
  labels:
    app: wordpress
spec:
  storageClassName: gp2  # Important: Must specify storage class
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wp-pv-claim
  labels:
    app: wordpress
spec:
  storageClassName: gp2  # Important: Must specify storage class
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
```
        
## Applying the Configuration
    
> kubectl apply -f pvcs.yaml -n ns-eks-course
> kubectl apply -f deploy-mysql.yaml -n ns-eks-course

    
