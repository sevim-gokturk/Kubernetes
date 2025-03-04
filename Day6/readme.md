# Kubernetes Secrets and ConfigMaps Hands-on Guide

This guide is designed for those who want to learn how to use **Secrets** and **ConfigMaps** in Kubernetes step by step. ConfigMaps and Secrets are effective ways to manage configuration data for your applications.

---

## 📌 1. What is a ConfigMap?

A **ConfigMap** is a Kubernetes object that contains configuration data. ConfigMaps store information such as environment variables, configuration files, or command-line arguments.

### 🛠 Step 1: Creating a ConfigMap

A ConfigMap can be created in two ways:

1. **Using a file**
2. **Using the command line**

#### **Method 1: Creating a ConfigMap Using a File**

First, create a YAML file:

```yaml
touch my-config.yaml
```

Then, add the following content to it:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
  namespace: default
data:
  database_url: "postgres://user:password@db:5432/mydb"
  log_level: "debug"
```

To create this ConfigMap, run the following command:

```bash
kubectl apply -f my-config.yaml
```

#### **Method 2: Creating a ConfigMap Using the Command Line**

```bash
kubectl create configmap my-config \
  --from-literal=database_url="postgres://user:password@db:5432/mydb" \
  --from-literal=log_level="debug"
```

---

## 🔑 2. What is a Kubernetes Secret?

A **Secret** is a Kubernetes object that allows you to securely store sensitive data (e.g., passwords, API keys). Unlike ConfigMaps, Secrets are stored in base64-encoded format.

### 🛠 Step 2: Creating a Secret

Secrets can also be created in two ways:

1. **Using a file**
2. **Using the command line**

#### **Method 1: Creating a Secret Using a File**

Create a YAML file:

```yaml
touch my-secret.yaml
```

Edit the content as follows:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
  namespace: default
type: Opaque
data:
  username: dXNlcm5hbWU=  # 'username' base64 encoded
  password: cGFzc3dvcmQ=  # 'password' base64 encoded
```

⚠ **Encode the values in base64 before adding them to the file:**

```bash
echo -n "username" | base64
echo -n "password" | base64
```

Create the Secret using:

```bash
kubectl apply -f my-secret.yaml
```

#### **Method 2: Creating a Secret Using the Command Line**

```bash
kubectl create secret generic my-secret \
  --from-literal=username=user123 \
  --from-literal=password=supersecret
```

---

## 🔗 3. Using ConfigMaps and Secrets

There are different ways to use ConfigMaps and Secrets inside a Pod:

1. **As an environment variable**
2. **As a file**

### **Using a ConfigMap**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
    - name: my-container
      image: nginx
      env:
        - name: DATABASE_URL
          valueFrom:
            configMapKeyRef:
              name: my-config
              key: database_url
```

### **Using a Secret**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
    - name: my-container
      image: nginx
      env:
        - name: SECRET_USERNAME
          valueFrom:
            secretKeyRef:
              name: my-secret
              key: username
```

---

## 🛠 4. Updating ConfigMaps and Secrets

To update a ConfigMap or Secret, you can delete the old one and create a new one:

```bash
kubectl delete configmap my-config
kubectl create configmap my-config --from-literal=database_url="new-url"
```

Alternatively, you can update it directly using:

```bash
kubectl edit configmap my-config
```

To update a Secret:

```bash
kubectl delete secret my-secret
kubectl create secret generic my-secret --from-literal=username=newuser
```

---

## 🚀 5. Viewing ConfigMaps and Secrets

To view a ConfigMap:

```bash
kubectl get configmap my-config -o yaml
```

To view a Secret:

```bash
kubectl get secret my-secret -o yaml
```

To decode a base64-encoded Secret value:

```bash
kubectl get secret my-secret -o jsonpath='{.data.username}' | base64 --decode
```

---

## 🎯 Conclusion

- **ConfigMaps** allow you to store configuration data for your applications.
- **Secrets** help you securely manage sensitive information.
- You can use these inside Pods as **environment variables** or **files**.
- Use `kubectl` commands to create, update, and manage ConfigMaps and Secrets efficiently.

