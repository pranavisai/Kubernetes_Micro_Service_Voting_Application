### Create namespace vote
```
kubectl create namespace vote
```
#### Check that the namespace is created
```
kubectl get namespaces
```

### Vote Service
```
apiVersion: v1
kind: Service
metadata:
  name: vote
spec:
  type: NodePort
  selector:
    app: vote
  ports:
    - port: 8080
      targetPort: 80
      nodePort: 31000
```
#### Create vote service
```
kubectl apply -f vote-service.yaml -n vote
```

### Vote deployment
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vote
  namespace: vote
spec:
  replicas: 1
  selector:
    matchLabels:
      app: vote
  template:
    metadata:
      labels:
        app: vote
    spec:
      containers:
      - name: vote
        image: dockersamples/examplevotingapp_vote
```
#### Create vote deployment
```
kubectl apply -f vote-deployment.yaml -n vote
```

### Result service 
```
apiVersion: v1
kind: Service
metadata:
  name: result
spec:
  type: NodePort
  selector:
    app: vote
  ports:
    - port: 8081
      targetPort: 80
      nodePort: 31001
```
#### Create result service
```
kubectl apply -f result-service.yaml -n vote
```

### Result deployment
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: result
  namespace: vote
spec:
  replicas: 1
  selector:
    matchLabels:
      app: result
  template:
    metadata:
      labels:
        app: result
    spec:
      containers:
      - name: result
        image: dockersamples/examplevotingapp_result
```
#### Create result deployment
```
kubectl apply -f result-deployment.yaml -n vote
```

### Worker deployment
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: worker
  namespace: vote
spec:
  replicas: 1
  selector:
    matchLabels:
      app: worker
  template:
    metadata:
      labels:
        app: worker
    spec:
      containers:
      - name: worker
        image: dockersamples/examplevotingapp_worker
```
#### Create worker deployment
```
kubectl apply -f worker-deployment.yaml -n vote
```

### Redis service
```
apiVersion: v1
kind: Service
metadata:
  name: redis
  namespace: vote
spec:
  type: ClusterIP
  selector:
    app: redis
  ports:
    - port: 6379
      targetPort: 6379
```
#### Create Redis service
```
kubectl apply -f redis-service.yaml -n vote
```

### Redis deployment
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
  namespace: vote
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:alpine
        volumeMounts:
        - name: redis-data
          mountPath: /data
      volumes:
      - name: redis-data
        emptyDir: {}
```
#### Create Redis deployment
```
kubectl apply -f redis-deployment.yaml -n vote
```

### DB service
```
apiVersion: v1
kind: Service
metadata:
  name: db
  namespace: vote
spec:
  type: ClusterIP
  selector:
    app: db
  ports:
    - port: 5432
      targetPort: 5432
```
#### Create DB service
```
kubectl apply -f db-service.yaml -n vote
```

### DB deployment
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: db
  namespace: vote
spec:
  replicas: 1
  selector:
    matchLabels:
      app: db
  template:
    metadata:
      labels:
        app: db
    spec:
      containers:
      - name: db
        image: postgres:15-alpine
        env:
        - name: POSTGRES_HOST_AUTH_METHOD
          value: "trust"
        volumeMounts:
        - name: db-data
          mountPath: /var/lib/postgresql/data
      volumes:
      - name: db-data
        emptyDir: {}
```
#### Create DB deployment
```
kubectl apply -f db-deployment.yaml -n vote
```

### Check if the deployments are running
```
kubectl get deployments -n vote
```

### Check if the services are running
```
kubectl get services -n vote
```
