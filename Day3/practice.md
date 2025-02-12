# Kubernetes Networking and Service Discovery

## Learning Outcomes

By the end of this training, students will be able to:

- Explain the benefits of grouping `Pods` with `Services`.
- Explore service discovery options in Kubernetes.
- Learn different types of Kubernetes Services.

## Outline

1. **Setting up the Kubernetes Cluster**
2. **Services, Load Balancing, and Networking in Kubernetes**

## Part 1 - Setting up the Kubernetes Cluster

- Launch a Kubernetes Cluster (Ubuntu 20.04, one master, one worker).(Day1/practice.md)
- Check if Kubernetes is running:
  ```bash
  kubectl cluster-info
  kubectl get no
  ```

## Part 2 - Services, Load Balancing, and Networking

### Defining and Deploying Services

1. **Create a folder for service files:**
```bash
mkdir service-lessons
cd service-lessons
```

2. **Create a `web-flask.yaml` deployment file:**
```yaml
apiVersion: apps/v1 
kind: Deployment 
metadata:
  name: web-flask-deploy
spec:
  replicas: 3 
  selector:  
    matchLabels:
      app: web-flask # deployment bu etiketi taşıyan podlarla eşleşir
  minReadySeconds: 10 #hazır olduğunu anlamamız için geçen süre
  strategy: # pod ekleme çıkarma güncelleme sırasında izlenecek yol
    type: RollingUpdate # sırayla güncelle ya da recreate ile aynı anda güncelle
    rollingUpdate:
      maxUnavailable: 1 
      maxSurge: 1 
  template: # bu kısımdan itibaren pod ve container detayları tanımlanır
    metadata:
      labels: # pod etikeleri burada
        app: web-flask
        env: front-end
    spec:
      containers:
      - name: web-flask-cont
        image: mefekadocker/web-flask:0.2 # docker hub repodan çekilen imajın adı
        ports:
        - containerPort: 8000 # container içindeki uygulamanın portu
```

3. **Deploy `web-flask.yaml`:**
```bash
   kubectl apply -f web-flask.yaml
 ```

4. **Check Pods and their IPs:**
```bash
   kubectl get pods -o wide
```
- We get an output like below.

```text
NAME                                READY   STATUS    RESTARTS   AGE   IP               NODE
web-flask-deploy-8649c67947-2w568   1/1     Running   0          9s    172.16.180.4   kube-worker-1   <none>           <none>
web-flask-deploy-8649c67947-db5vm   1/1     Running   0          9s    172.16.180.6   kube-worker-1   <none>           <none>
web-flask-deploy-8649c67947-hdjd8   1/1     Running   0          9s    172.16.180.5   kube-worker-1   <none>           <none>
```

In the output above, for each Pod the IPs are internal and specific to each instance. If we were to redeploy the application, then each time a new IP will be allocated.

5. **Test internal networking using a `busybox` pod:**
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: busybox-test
     namespace: demo
   spec:
     containers:
     - name: busybox
       image: radial/busyboxplus:curl
       command: ['sh','-c','while true;do sleep 3600;done']
   ```

6. **Deploy `busybox` pod and check connectivity:**
```bash
   kubectl apply -f test.yaml
   kubectl exec -it busybox-test -- curl <POD-IP>:8000 
   kubectl exec -it busybox-test -- sh
   / # ping <POD-IP>
```

### Creating a ClusterIP Service

1. **Create `web-svc.yaml`:**
```yaml
apiVersion: v1
kind: Service   
metadata:
  name: web-flask-svc
  labels:
    app: web-flask
spec:
  type: ClusterIP  
  ports:
  - port: 3000  # clusterIP servis portu
    targetPort: 8000 # container yayınını yakalamak için hedef port
  selector:
    env: front-end # label env: front-end olan podlar eşlenecek
```

2. **Deploy the Service:**
```bash
   kubectl apply -f web-svc.yaml
 ```

3. **List Services:**
```bash
   kubectl get svc -o wide
```
```text
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE     SELECTOR
kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP                  4h39m   <none>
web-flask-svc   ClusterIP   10.98.173.110   <none>        3000/TCP                 28m     app=web-flask
kube-dns        ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP,9153/TCP   4h39m   k8s-app=kube-dns
```
- Display information about the `web-flask-svc` Service.

```bash
kubectl describe svc web-flask-svc
```

```text
Name:              web-flask-svc
Namespace:         default
Labels:            app=web-flask
Annotations:       <none>
Selector:          app=web-flask
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4 
IP:                10.105.60.230
IPs:               10.105.60.230
Port:              <unset>  3000/TCP
TargetPort:        5000/TCP
Endpoints:         172.16.180.5:5000,172.16.180.6:5000
Session Affinity:  None
Events:            <none>
```

4. **Test Service connectivity inside the cluster:**
```bash
   kubectl exec -it busybox-test -- sh
   / # curl web-flask-svc:3000
```

### Creating a NodePort Service

1. **Modify `web-svc.yaml`:**
- Change the service type of web-flask-svc service to NodePort to use the Node IP and a static port to expose the service outside the cluster. So we get the yaml file below.

```yaml
apiVersion: v1
kind: Service   
metadata:
  name: web-flask-svc
  labels:
    app: web-flask
spec:
  type: NodePort  
  ports:
  - port: 3000  
    targetPort: 8000
  selector:
    env: front-end 
```

2. **Apply changes and list Services:**
```bash
   kubectl apply -f web-svc.yaml
   kubectl get svc -o wide
```

3. **Access the application via Node IP and NodePort:**
```bash
   http://<public-node-ip>:<node-port>
```