## Setting up dynamic provisioning in EKS cluster with EBS volumes

**Note:** The cluster used for this practice is the one created with `eksctl` in the `00-cluster-setup` folder.

Before starting, you must create the cluster and update your `kubeconfig`

Prerequisites:
- AWS CLI installed and configured
- kubectl installed
- eksctl installed
- helm installed

#### 1- Associate an IAM OIDC Provider with the Cluster

```bash
eksctl utils associate-iam-oidc-provider --region us-east-1 --cluster my-cluster --approve
```
#### 2- Create an IAM Service Account for the EBS CSI Driver

Create an IAM service account with permissions to manage Amazon EBS volumes. This example:
- Specifies the namespace `kube-system`, cluster name `my-cluster` and the region `us-east-1`.
- Attaches the **AmazonEBSCSIDriverPolicy**, which provides the necessary permissions for the EBS CSI driver.

**Note**: Copy the command and paste it in an editor if you have some modifications on the cluster name or the region for example. After modifications you can run it in your terminal.

```bash
eksctl create iamserviceaccount \
  --name ebs-csi-controller-sa \
  --namespace kube-system \
  --cluster my-cluster \
  --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
  --approve \
  --region us-east-1
```
#### 3- Install the Amazon EBS CSI Driver
Install the EBS CSI driver, which allows Kubernetes to manage EBS volumes dynamically. 

This command applies the configuration files from the official AWS EBS CSI driver GitHub repository.
```bash
kubectl apply -k "github.com/kubernetes-sigs/aws-ebs-csi-driver/deploy/kubernetes/overlays/stable/ecr/?ref=release-1.20"
```
#### 4- Verify EBS CSI Controller Pods
Verify that the EBS CSI controller pods are running in the kube-system namespace. This step ensures that the driver was installed correctly.
```bash
kubectl get pods -n kube-system -l app=ebs-csi-controller
```
#### 5- Verify DaemonSets and Deployments for the EBS CSI Driver
Verify that the EBS CSI driver DaemonSets and Deployments were successfully created. 

These resources should be running in the **kube-system** namespace.
```bash
kubectl get daemonset -n kube-system | grep ebs-csi
kubectl get deployment -n kube-system | grep ebs-csi
```
#### 6- Verify Available Storage Classes
List the storage classes to ensure the EBS CSI driver storage class is available. 

This storage class will be used to create dynamically provisioned volumes.
```bash
kubectl get storageclass
```
You can see the default storage class (**gp2**)

We will deploy a PVC using this default storage class and deploy a Pod that will comsume that PVC.

### LAB: Using dynamic storage with EBS to deploy a pod with persistent storage

This lab demonstrates how to create a **PersistentVolumeClaim (PVC)** using EBS with the default **gp2 StorageClass** in AWS EKS.

After enabling dynamic provisioning in EKS, you can create a PVC and deploy a pod that will consume that volume.

1. Create a manifest for the PVC and the pod (`ebs-pvc-pod.yaml`). 

The content should look like the following:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ebs-pvc
spec:
  accessModes:
    - ReadWriteOnce  # Only one node can mount the volume
  resources:
    requests:
      storage: 5Gi  # Request 5GB of storage
  storageClassName: gp2  # Default StorageClass for EBS in AWS
---
## Pod that will consume the PVC
apiVersion: v1
kind: Pod
metadata:
  name: pod-ebs
spec:
  containers:
  - name: app
    image: busybox
    command: [ "sh", "-c", "echo 'Hello from EBS!' > /data/test.txt && sleep 3600" ]
    volumeMounts:
    - mountPath: "/data"
      name: storage
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: ebs-pvc
```
2. Apply your file in the cluster

```bash
kubectl apply -f ebs-pvc-pod.yaml
```
Verify that the PVC status is **Bound** and that the pod is running and using the volume
```bash
kubectl get pvc
kubectl get pod
kubectl exec -it pod-ebs -- cat /data/test.txt
```
If the pod is recreated on the same node, it should still have access to the data.

**Note:** In AWS EKS, **EBS does not support** **RWX(ReadWriteMany)**. It is limited to **RWO(ReadWriteOnce)** It means the volume created by ebs can only be mounted to one node. 

To allow many nodes to mount a volume, you need to use another service like **Amazon EFS** that support **RWX**.
