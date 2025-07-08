# Deploying StatefulSets

## Objectives

By completing this lab, you will:

- Understand the purpose and benefits of StatefulSets in Kubernetes/OpenShift
- Learn how to create and apply a StatefulSet manifest
- Verify persistent storage and network identity for stateful workloads like databases

## Prerequisites

Before starting, make sure you have:

- A working Kubernetes/OpenShift cluster (Minikube, Kind, CRC, or OpenShift Local)
- `kubectl` or `oc` CLI tools configured correctly
- Basic knowledge of Kubernetes objects like Pods, PVCs, and Services
- A `StorageClass` configured in your cluster (for example: `standard` in Minikube)

###  Task 1: Writing a StatefulSet Manifest

We’ll start by writing a simple `StatefulSet` YAML manifest for MySQL.

Create a file named **`mysql-statefulset.yaml`**:

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: "mysql"
  replicas: 2
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "password"
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: mysql-persistent-storage
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "standard"
      resources:
        requests:
          storage: 1Gi
````

Now validate the manifest:

```bash
kubectl apply -f mysql-statefulset.yaml --dry-run=client
```

You should not see any validation errors.

### Task 2: Deploy the Stateful Application

Apply the StatefulSet:

```bash
kubectl apply -f mysql-statefulset.yaml
```

Now create a **headless Service** to support stable network identities:

Create a new file `mysql-service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  clusterIP: None
  ports:
  - port: 3306
    name: mysql
  selector:
    app: mysql
```

Apply the service:

```bash
kubectl apply -f mysql-service.yaml
```

Check the deployed resources:

```bash
kubectl get statefulset,pods,pvc
```

You should see 2 Pods and 2 PVCs created and in use.

### Task 3: Verifying Storage and DNS Identity

1. **Check Pod Names**:

```bash
kubectl get pods -l app=mysql -o wide
```

Expected: Pods like `mysql-0` and `mysql-1`.

2. **Test Persistence**:

```bash
kubectl exec -it mysql-0 -- mysql -uroot -ppassword -e "CREATE DATABASE lab_test;"
kubectl delete pod mysql-0
kubectl exec -it mysql-0 -- mysql -uroot -ppassword -e "SHOW DATABASES;"
```

Expected: `lab_test` should still exist.

3. **Check DNS Resolution**:

```bash
kubectl run -it --rm --image=busybox:1.28 test --restart=Never -- nslookup mysql
```

Expected: You should see DNS entries resolving for `mysql-0.mysql`, `mysql-1.mysql`, etc.

## Troubleshooting

* **PVC Stuck in Pending**:

  Run:

  ```bash
  kubectl get storageclass
  ```

  For Minikube:

  ```bash
  minikube addons enable storage-provisioner
  ```

* **Pod CrashLoopBackOff or Fail to Start**:

  Check logs:

  ```bash
  kubectl logs mysql-0
  ```

  Make sure the environment variable `MYSQL_ROOT_PASSWORD` is set properly.

## Cleanup

When you're done, clean up the environment:

```bash
kubectl delete -f mysql-statefulset.yaml -f mysql-service.yaml
```
