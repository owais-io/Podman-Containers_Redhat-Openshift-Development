# Creating and Managing Pods


## Task 1: Creating a Pod

### Subtask 1.1: Create a Basic Pod

You can create a pod in Podman using the `podman pod create` command. The following command creates a new pod named `demo-pod` and maps port `8080` on the host to port `80` in the pod:

```bash
podman pod create --name demo-pod -p 8080:80
````

To verify that the pod was created:

```bash
podman pod list
```

**Expected Output:**

```
POD ID    NAME      STATUS    CREATED         INFRA ID      # OF CONTAINERS
abc123    demo-pod  Created   2 minutes ago   def456        1
```

### Subtask 1.2: Add a Container to the Pod

Run an Nginx container within the pod:

```bash
podman run -d --pod demo-pod --name nginx-container docker.io/library/nginx:alpine
```

To see all containers within the pod:

```bash
podman ps --pod
```

You should see both the infra container and the `nginx-container`.

## Task 2: Running Multiple Containers in a Pod

### Subtask 2.1: Add a Sidecar Container

Let’s add a Redis container to the same pod:

```bash
podman run -d --pod demo-pod --name redis-container docker.io/library/redis:alpine
```

To verify that both containers are in the pod:

```bash
podman pod inspect demo-pod | jq '.Containers[].Names'
```

**Expected Output:**

```
"nginx-container"
"redis-container"
```

### Subtask 2.2: Verify Shared Network

Containers inside a pod share the same network namespace. You can verify this by accessing one container from another.

First, exec into the Nginx container:

```bash
podman exec -it nginx-container sh
```

Then ping the Redis container:

```sh
ping redis-container
```

The Redis container should be reachable using the hostname `redis-container`.

## Task 3: Inspecting Pod Networking and Volumes

### Subtask 3.1: Network Inspection

To check the pod’s IP address and related network settings:

```bash
podman pod inspect demo-pod | jq '.InfraConfig.NetworkOptions'
```

To see the port mappings:

```bash
podman port demo-pod
```

### Subtask 3.2: Create Shared Volume

Create a shared volume and mount it to two containers in the pod:

```bash
podman volume create shared-vol
```

Now run two containers that both mount the shared volume:

```bash
podman run -d --pod demo-pod --name nginx2 \
  -v shared-vol:/data docker.io/library/nginx:alpine

podman run -d --pod demo-pod --name redis2 \
  -v shared-vol:/data docker.io/library/redis:alpine
```

To verify that the volume is shared:

```bash
podman exec -it nginx2 touch /data/testfile
podman exec -it redis2 ls /data
```

**Expected Result:** You should see `testfile` inside the Redis container as well.

## Troubleshooting Tips

* If a pod or container fails to start, inspect logs:

```bash
podman logs <container_name>
```

* For network-related issues, ensure the necessary ports are open:

```bash
sudo firewall-cmd --list-ports
```

* Clean up failed pods:

```bash
podman pod rm -f demo-pod
```

## Cleanup

To remove the pod and associated containers:

```bash
podman pod rm -f demo-pod
```

To remove the shared volume:

```bash
podman volume rm shared-vol
```
