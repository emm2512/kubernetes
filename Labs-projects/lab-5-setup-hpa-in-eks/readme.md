# Setup HPA in AWS EKS cluster and deploy a sample application

## 1. Deploy the metric server
Apply the manifest to deploy the Kubernetes metric server from the official documentation (use version v0.5.0)

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.5.0/components.yaml
```
Verify the status of the metric server pods inn the `kube-system` namespace

```bash
kubectl get pods -n kube-system
```
## 2. Deploy a sample application with the hpa defined for the deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 2
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

- **cpu-percent=50**: The target average CPU utilization
- **min=2**: The lower limit for the number of pods that can be set by the autoscaler
- **max=5**: The upper limit for the number of pods that can be set by the autoscaler

Verify the hpa:
```bash
kubectl get hpa
```
**WARNING**: You probably will see <unknown>/50% for 1-2 minutes and then you should be able to see 0%/50%

## 4. Generate load to trigger the autoscaling

Open another Gitbash terminal run a load generator pod

```bash
kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- sh 
#Execute a while loop to continue getting http://<pod-IP>
while true; do wget -q -O - http://nginx-service; done
```

In the previous terminal, watch the HPA with the following command
```bash
kubectl get pods
kubectl top pods
kubectl get hpa -w
```
Stop the load generation in the second terminal with CTRL + C and watch the HPA behaviour

```bash
kubectl get pods
kubectl top pods
kubectl get hpa -w
```

# Clean Up

Delete all the objects created.
