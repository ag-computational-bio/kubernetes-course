# Kubernetes 101 - Getting started
 
### 0 - Installing and configuring kubectl
 
The primary source for the Hands-On session is the [official kubernetes documentation](https://kubernetes.io/docs/home/). 
 
For getting started first the commandline application `kubectl` must be installed. A detailed installation guideline can be found [here](https://kubernetes.io/docs/tasks/tools/). While not strictly necessary it is recommended to configure bash autocompletion.
 
Login to the Rancher WebUI [here](https://rancher.biokube.org) with your designated username and password and select the `denbi-course` cluster. Download the KubeConfig file and move it to the location `~/.kube/config`.
 
Test the configuration by running: `kubectl get pods -n <namespace>`. 
A working configuration is indicated by a message like this: `No resources found in <namespace> namespace.`
 
### 1 - Running simple pods via YAML
 
Pods are the most basic resource for running workloads in a kubernetes cluster.
Every resource for kubernetes can be described as YAML file. For example a simple pod can look like this:
 
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "echo Hello Kubernetes! && sleep 3600"]
```
 
- Run this pod usind `kubectl apply -f <YAML> -n <namespace>`.
- Check if the pod is running.
- Use `kubectl logs <podname> -n <namespace>` to get the logs of the pod.
- Login to the pod via an interactive session `kubectl exec -it <podname> -n <namespace> -- sh`

Tip: You can change your context for a default namespace with `kubectl config set-context --current --namespace=<insert-namespace-name-here>`. If this is set to your specific namespace the `-n <namespace>` parameter can be omitted.

### 1 - (Excursus) Imperative use of kubectl

Most kubectl commands can either be used declarative via a YAML file or optionally imperative (similar to docker).

For example you can run the busybox image like this:

`kubectl run busybox --image=busybox --command -- sh -c "echo Hello Kubernetes! && sleep 3600"`

This is especially useful with the `--dry-run=client` and `-o=yaml` parameters, which allow you to create a basic YAML file via kubectl without referencing the documentation.

Example: 

`kubectl run busybox --image=busybox --dry-run=client -o=yaml --command -- sh -c "echo Hello Kubernetes! && sleep 3600" > busybox.yaml`

This procedure can also be used to create an interactive pod session on the fly:

`kubectl run -i --tty busybox --image=busybox --restart=Never`


For additional tips and tricks please visit the official kubernetes [kubectl cheatsheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

### 2 - Jobs

Jobs are kubernetes resources that manage pods intended to run to (successful) completion (like calculations or tasks). This is similar to regular batch processing engines like SLURM or SGE.

A simple example for a job looks like this:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
```

This job uses the `perl` container to calculate 2000 decimal places of Pi.

- Excecute the job
- Examine the created pods
- You can get a more detailed status description with `kubectl describe jobs/pi`
- Examine the logs after the pod is finished.

The finished pod and job will stay forever. You can change this behaviour if you specify a time to live (TTL) with `ttlSecondsAfterFinished: 100` in the spec section of the job. This results in automatic deletion of the job and pod after 100 seconds.

- Delete the old job
- Spawn a new job with a TTL deadline

Tip: Jobs can also be created imperatively:

`kubectl create job pi --image=perl -- perl -Mbignum=bpi -wle "print bpi(2000)"`


 
 
 
