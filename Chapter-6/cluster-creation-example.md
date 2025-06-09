# Creating a GKE Cluster and Deploying an Application  
This section provides a step-by-step guide on creating a GKE cluster and deploying an application. It focuses on configuring the cluster and ensuring seamless deployment within the Kubernetes environment.


1.	Create a GKE Cluster
```bash
gcloud container clusters create demo-cluster \
  --zone us-central1-a \
  --num-nodes 2
  --project <your-kubernetes-project-name>
```

2.	Login to Cloud Shell and get the cluster credentials
```bash
gcloud container clusters get-credentials demo-cluster 
  --zone us-central1-a 
  --project <your-kubernetes-project-name>
```
3.	Deploy Application
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web-container
        image: nginx:latest
```
Then apply it through the kubectl command

```bash 
kubectl apply -f deployment.yaml 
```
4.	Create a Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
```
Then apply it through the kubectl command
```bash 
kubectl apply -f service.yaml 
```
5.	Enable Horizontal Pod Autoscaling

```bash 
kubectl autoscale deployment demo-app --cpu-percent=50 --min=1 --max=5 
```

The above command will keep on launching newer Pods with the CPU utilization across all the Pods breaches 50% and it will scale-down the pods when the overall CPU Utilization across all the Pods go below 50%. Minimum number of Pods it will scale down to is 1 and the Maximum Number of Pods it will scale up to is 5

6.	Enable Cluster Autoscaling

```bash
gcloud container clusters update demo-cluster \
    --enable-autoscaling \
    --min-nodes=1 \
    --max-nodes=10
```
