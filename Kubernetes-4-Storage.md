# Kubernetes 101 - Storage
 
### 1 - Volumes

[Volumes & PVC (official docs)](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)

Kubernetes supports multiple storage types. The most common storage type is a volume. Volumes depend on external storage addons and are provisioned via a Persistent Volume Claim (PVC)

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
spec:
  storageClassName: cinder-csi
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

[Configmaps (official docs)](https://kubernetes.io/docs/concepts/configuration/configmap/) / 
[Secrets (official docs)](https://kubernetes.io/docs/concepts/configuration/secret/)

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

[Official docs section](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#define-container-environment-variables-using-configmap-data)

Secrets and configmaps can be used to directly change the environment variables of a container. This is very useful when configuring containers, almost all configurable containers have the option to be configured via environment variables.

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

Create a pod with the above YAML, connect to it interactively and check that the environment variables `SomeName`, `SECRET_USERNAME` and `SECRET_PASSWORD` are set.

### 4 - Secrets / Configmaps as volumes/files

[Official docs section](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#populate-a-volume-with-data-stored-in-a-configmap)

Configmaps and secrets can not only be used to populate environment variables, they can also be used as volume mounts directly. This can be used to overwrite existing files inside a container which is especially useful for small configuration files.

Caveat: Please be aware that secrets and configmaps (and in fact all kubernetes resources) are stored in etcd which imposes a size limit of 1MB that cannot be exceeded.

A simple example for a configmap based volume-mount can be found here:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: busybox
      command: [ "/bin/sh", "-c", "ls /etc/config/" ]
      volumeMounts:
      - name: config-volume
        mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        # Provide the name of the ConfigMap containing the files you want
        # to add to the container
        name: special-config
  restartPolicy: Never
```



 
 
 
