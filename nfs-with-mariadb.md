# Cluster Details 

```
root@k8s-manager-azmin:~/database# kubectl get nodes -o wide
NAME                STATUS   ROLES           AGE     VERSION    INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION    CONTAINER-RUNTIME
k8s-manager-azmin   Ready    control-plane   7d3h    v1.29.10   192.168.10.242   <none>        Ubuntu 22.04.5 LTS   5.15.0-1068-kvm   docker://27.3.1
k8s-worker1-azmin   Ready    <none>          6d23h   v1.29.10   192.168.10.60    <none>        Ubuntu 22.04.5 LTS   5.15.0-1068-kvm   docker://27.3.1
k8s-worker2-azmin   Ready    <none>          6d23h   v1.29.10   192.168.10.245   <none>        Ubuntu 22.04.5 LTS   5.15.0-1068-kvm   docker://27.3.1

```

I'm using `k8s-manager-azmin` this node as NFS Server.

# Step 1: Set Up the NFS Server

```
sudo apt update
sudo apt install nfs-kernel-server -y
```

# Create a Directory for NFS Exports

```
sudo mkdir -p /srv/nfs/mariadb
```

# Set Permissions

```
sudo chown -R nobody:nogroup /srv/nfs/mariadb
sudo chmod 777 /srv/nfs/mariadb
```

# Configure NFS Exports

```
sudo nano /etc/exports
```

Add the following line to share the directory with your Kubernetes nodes:

```
/srv/nfs/mariadb *(rw,sync,no_subtree_check,no_root_squash)
```
 > Note: The * allows access from any IP. For more security, you can replace * with the specific IP addresses of your manager and worker nodes, like so:
> ```
> 192.168.10.242(rw,sync,no_subtree_check,no_root_squash)
> 192.168.10.60(rw,sync,no_subtree_check,no_root_squash)
> 192.168.10.245(rw,sync,no_subtree_check,no_root_squash)
> ```

# Apply NFS Exports Configuration

```
sudo exportfs -a
```
# Start the NFS Service

```
sudo systemctl restart nfs-kernel-server
```


# Step 2: Create a Persistent Volume (PV) and Persistent Volume Claim (PVC)

* Create a Persistent Volume YAML File `mariadb-pvc.yaml`

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mariadb-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteMany
  nfs:
    path: /srv/nfs/mariadb
    server: 192.168.10.242  # Your NFS server IP (manager node)

---

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mariadb-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Gi
```


`kubectl apply -f mariadb-pvc.yaml`


* Deploy MariaDB `mariadb-deployment.yaml`

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mariadb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mariadb
  template:
    metadata:
      labels:
        app: mariadb
    spec:
      containers:
      - name: mariadb
        image: mariadb:latest
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "your_root_password"  # Change this to a strong password
        ports:
        - containerPort: 3306
        volumeMounts:
        - mountPath: /var/lib/mysql
          name: mariadb-storage
      volumes:
      - name: mariadb-storage
        persistentVolumeClaim:
          claimName: mariadb-pvc

---

apiVersion: v1
kind: Service
metadata:
  name: mariadb
spec:
  ports:
  - port: 3306
    targetPort: 3306
  selector:
    app: mariadb
  type: ClusterIP

```

`kubectl apply -f mariadb-deployment.yaml`

* Verify the Setup
```
kubectl get pods
kubectl get svc
```









