### Install postgresql

1) Apply manifests

a) deployment.yaml
```
# Kubernetes API version
apiVersion: apps/v1
# Deployment object
kind: Deployment
metadata:
  # The name of the Deployment
  name: postgres
spec:
  # Replicas for this Deployment
  replicas: 1
  selector:
    # labels the pods
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        # The label the pods created from the pod template should have
        app: postgres
    spec:
      containers:
        # The container name to execute pods
        - name: postgres
          # pull postgresimage from docker hub
          image: postgres
          ports:
            # Assign ports to expose container
            - containerPort: 5432
          envFrom:
            # Load the environment variables/PostgresSQL credentials
            - configMapRef:
                # This should be the ConfigMap name created ealier
                name: db-secret-credentials
          volumeMounts:
            # The volume mounts  for the container
            - mountPath: /var/lib/postgres/data
              name: db-data
      # Volumes attached to the pod
      volumes:
        - name: db-data
          persistentVolumeClaim:
            # reference the PersistentVolumeClaim
            claimName: db-persistent-volume-claim
```
b) configmap.yaml
```
apiVersion: v1
# Kind for kubernets ConfigMap
kind: ConfigMap
metadata:
  # Name your ConfigMap
  name: db-secret-credentials
  labels:
    app: postgres
data:
  # User DB
  POSTGRES_DB: postgres_DB
  # Db user
  POSTGRES_USER: postgres
  # Db password
  POSTGRES_PASSWORD: postgres
```
c) pv.yaml
```
apiVersion: v1
# Kind for volume chain
kind: PersistentVolume
metadata:
  # Name the persistent chain
  name: postgresdb-persistent-volume
  # Labels for identifying PV
  labels:
    type: local
    app: postgres
spec:
  storageClassName: manual
  capacity:
    # PV Storage capacity
    storage: 8Gi
  # A db can write and read from volumes to multiple pods
  accessModes:
    - ReadWriteMany
  # Specify the path to persistent the volumes  
  hostPath:
    path: "/data/db"
```
d) pvc.yaml
```
apiVersion: v1
# define a resource for volume chain
kind: PersistentVolumeClaim
metadata:
  # Name the volume chain
  name: db-persistent-volume-claim
spec:
  storageClassName: manual
  accessModes:
    # Allow ReadWrite to multiple pods
    - ReadWriteMany
  # PVC requesting resources
  resources:
    requests:
      # the PVC storage
      storage: 8Gi
```
e) service.yaml
```
apiVersion: v1
# Kind for service
kind: Service
metadata:
  # Name your service
  name: postgres
  labels:
    app: postgres
spec:
  # Choose how to expose your service
  type: NodePort
  ports:
    # The port number to expose the service
    - port: 5432
  # Pod to route service traffic  
  selector:
    app: postgres
```

## Создайте директорию postgresql, поместите туда все манифесты указанные выше и примените команду:
```
kubectl apply -f postgresql/
```
