# Kubernetes 101 - Deployments
 
[Deployment (official docs)](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

### 1 - Basic deployment example

Deployments are Kubernetes workloads for permanently running applications (e.g. Websites or Services). They are build on top of ReplicaSets and enable replication and rollout strategies.

A basic deployment using the nginx image looks like this:

```yaml
apiVersion: apps/v1
kind: Deployment
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
        image: nginx:1.21.4
        ports:
        - containerPort: 80
```

Note: The used container exposes an internal nginx webserver / proxy.

- Create the deployment
- Inspect the created pods
- What happens if you delete one of these pods manually ?


### 2 - Deployment rollouts and scaling

Because deployments are based on ReplicaSets they offer more advanced rollout strategies when the underlying specification has changed.
This enables an application to always have a minimum availability and enables a change history and undo capabilities.

The rollout status can be monitored with the `kubectl rollout status deployment/<name>` command.
A deployment can be restarted with `kubectl rollout restart deployment/<name>`.
`kubectl rollout history deployment/<name>` inspects the history of previous changes and `kubectl rollout undo deployment/<name>` allows for reverting to a previous state (use `--to-revision=3` for a specific revision number).

`kubectl scale deployment/<name> --replicas=5` is used to change the current replica-count of a deployment.

- Scale your deployment down (from three replicas to two)
- Update your nginx deployment from `nginx:1.14.2` to the current version `nginx:1.21.4`.
- Watch the rollout status and wait till it finishes
- Inspect the rollout history
- Undo your previous change