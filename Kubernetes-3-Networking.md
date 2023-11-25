# Kubernetes 101 - Networking
 
### 1 - Services

[Services (official docs)](https://kubernetes.io/docs/concepts/services-networking/service/)

In Kubernetes pods may change their internal IP at any time. This problem can be solved via services.
Services introduce a stable IP and hostname for a collection of pods identified by selectors.

A basic service may look like this:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  # Define the target selector
  selector:
    app: nginx
  # Define the ports & protocol
  ports:
    - name: http
      protocol: TCP
      port: 80
```

This creates a service that targets the pods of the nginx-deployment from the previous example.
Note: The ports section accepts multiple ports. The `port` entry specifies the exposed port of the service and the targeted port in the container.
You can separate the targeted and exposed port by specifying an additional `targetPort`.
E.g.
```
...
ports:
  - name: http
    protocol: TCP
    port: 80
    targetPort: 8080
...
```

- Create a service for your nginx deployment.
- Create an interactive busybox pod (`kubectl run -i --tty busybox --image=busybox --restart=Never`)
- Test the service dns-entry and internal ip with wget (service ip = `kubectl get service`, `wget -O - nginx-service` / `wget -O - <ip>`)

### 2 - Ingress

[Ingress (official docs)](https://kubernetes.io/docs/concepts/services-networking/ingress/)

While services primarily manage internal address resolution between different workloads inside the kubernetes cluster, Ingresses manage the communication with the external world / Internet.

A basic ingress example is shown here:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: demo.clum.gi.denbi.de
    http:
      paths:
      - path: /
        backend:
          serviceName: nginx-service
          servicePort: 80
```

This Ingress routes the publicly available subdomain `demo.clum.gi.denbi.de` to the nginx service / deployment.

Note: The hostname has to be unique -> please specify your own hostname. To prevent unnecessary clashes use: `<SOMETHING UNIQUE>.clum.gi.denbi.de` where `<SOMETHING>` is any arbitrary unique string.


### 2 - Ingress with TLS

The Kubernetes cluster has an automated TLS addon (letsencrypt) installed.
You can secure your nginx automatically by specifying annotations:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  annotations:
    kubernetes.io/ingress.class: "nginx"    
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  tls:
  - hosts:
    - demo.clum.gi.denbi.de
    secretName: demo-course-secret-tls
  rules:
  - host: demo.clum.gi.denbi.de
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80
´´´







 
 
 
