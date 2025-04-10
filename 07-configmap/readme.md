**Note**: This practice can be done on **Killercoda kubernetes playgrounds**

## The configMap resource

It is a good practice is to keep your application code separate from your environments (dev, test, qa, prod) configuration. 

A ConfigMap is an object that allows you to store non-confidential configuration data as key-value pairs. With ConfigMaps, you can store configuration settings like:  connection strings, public credentials, hostnames, and URLs.

### How to create a configmap

You can create a configmap from the command line

- Using literal values with the option `--from-literal` 
Example
```bash
kubectl create configmap my-config1 --from-literal port=3090 
kubectl get cm
kubectl describe cm my-config1
kubectl delete cm my-config1
```
- Using files and directories with the option `--from-file` to pass the filename or the directory 
```bash
vi app.configfile
```
The content should look like the following
```bash
port=2000
environment=staging
language=english
```

Create a configmap with this file
```bash
kubectl create configmap my-config2 --from-file app.configfile
kubectl get cm
kubectl describe cm my-config2
kubectl delete cm my-config2
```
- Using a manifest
```bash
vi 01-my-configmap.yaml
```

The content should look like the following
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-configmap
data:
  mydata: hello_world
```
Create the ConfigMap using the yaml 
```bash
kubectl create -f 01-my-configmap.yaml
kubectl get cm
kubectl describe cm my-configmap
```

### How to use the configmap in pods or deployments

There are four different ways that you can use a ConfigMap to configure a container inside a Pod:

- As environment variables for the container
- As a read-only Volume for the application to read
- Inside a container command and args
- Write a program that will run inside the Pod and use the Kubernetes API to read the ConfigMap

Here, we will show how to use the 2 first methods

#### 1- Using Environment Variables: 
Example 1: Let’s create a Pod (`02-my-configmap-pod.yaml`) that will consume the data from the configmap called my-configmap (`01-my-configmap.yaml`)

Create the pod using the content in `02-my-configmap-pod.yaml` file
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-configmap-pod1
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
      - containerPort: 80
      env:
        - name: cm
          valueFrom:
            configMapKeyRef:
              name: my-configmap
              key: mydata
```

```bash
kubectl create -f 01-my-configmap-pod1.yaml
kubectl get pods
```
Get into the pod and print all the environment variable
```bash
kubectl exec -it my-configmap-pod1 -- printenv
```
You can see the data from the configmap is set as environment variable in the pod.


#### 2- Using volumes

Let’s create a Pod (`03-pod-configmap-volume.yaml`) that will consume the data from the configmap (`01-my-configmap.yaml`) using volumes.

Create the  pod using the content in the `03-pod-configmap-volume.yaml` file

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-configmap-volume
spec:
  containers:
    - name: nginx
      image: nginx
      volumeMounts:
        - name: config-volume
          mountPath: /config
          readOnly: true
      ports:
      - containerPort: 80
  volumes:
    - name: config-volume
      configMap:
        name: my-configmap
```
```bash
kubectl create -f 03-pod-configmap-volume.yaml
kubectl get cm
kubectl get pods

#If you are not on a Linux server, the following commands might throw a No such file or directory error.
#Just make sure you understand the concept.

kubectl exec -it pod-configmap-volume -- ls /config

kubectl exec -it pod-configmap-volume -- cat /config/mydata
```

You can learn more on configmaps using the kubernetes official documentation