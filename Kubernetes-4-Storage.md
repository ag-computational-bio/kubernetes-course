# Kubernetes 101 - Storage
 
### 1 - Volumes

Kubernetes supports multiple storage types. The most common storage type is a volume. Volumes depend on external storage addons and are provisioned via a Persistent Volume Claim (PVC)

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
spec:
  storageClassName: longhorn
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

You can use the PVC like this:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ubuntu-deployment
  labels:
    app: ubuntu
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ubuntu
  template:
    metadata:
      labels:
        app: ubuntu
    spec:
      containers:
      - name: ubuntu
        image: debian
        command: [ "sleep" ]
        args: [ "infinity" ]
        volumeMounts:
        - mountPath: /mnt
          name: storage
      volumes:
      - name: storage
        persistentVolumeClaim:
          claimName: task-pv-claim
```

The `infinity sleep` is needed to keep the container alive (similar to docker). A succeeding pod in a deployment is considered a failure and results in rescheduling and crashloops. 

### 2 - Secrets / Configmaps

Example for a secret:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  username: YWRtaW4=
  password: MWYyZDFlMmU2N2Rm
```

Example for a configmap:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: game-demo
data:
  # property-like keys; each key maps to a simple value
  player_initial_lives: "3"
  ui_properties_file_name: "user-interface.properties"

  # file-like keys
  game.properties: |
    enemy.types=aliens,monsters
    player.maximum-lives=5    
  user-interface.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true

```

### 3 - Secrets / Configmaps as ENV-vars


```yaml
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
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
        image: nginx:1.7.9
        ports:
        - containerPort: 80
        env:
          - name: SomeName
            value: SomeValue
          - name: SECRET_USERNAME
            valueFrom:
              secretKeyRef:
                name: mysecret
                key: username
          - name: SECRET_PASSWORD
            valueFrom:
              secretKeyRef:
                name: mysecret
                key: password
```




 
 
 
