
# Create a PersistentVolume (PV) and PersistentVolumeClaim (PVC)

```
# mongo-pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongo-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data/mongo"  # Replace with your desired path
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongo-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

`kubectl apply -f mongo-pv.yaml`

# Deploy MongoDB

```
# mongo-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongodb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
        - name: mongodb
          image: mongo:latest
          ports:
            - containerPort: 27017
          volumeMounts:
            - mountPath: /data/db
              name: mongo-storage
      volumes:
        - name: mongo-storage
          persistentVolumeClaim:
            claimName: mongo-pvc

```

`kubectl apply -f mongo-deployment.yaml`


# Expose MongoDB

```
# mongo-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: mongodb-service
spec:
  type: ClusterIP
  ports:
    - port: 27017
      targetPort: 27017
  selector:
    app: mongodb
```

`kubectl apply -f mongo-service.yaml`


# Verify the Deployment


```
kubectl get pods
kubectl get pvc
kubectl get service mongodb-service
```





