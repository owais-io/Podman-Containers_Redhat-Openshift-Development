# Networking in Containers

## Task 1: List Podman Networks

### Subtask 1.1: Check Available Networks

To list all available networks:

```bash
podman network ls
````

**Expected Output:**

```
NETWORK ID    NAME        DRIVER
2f259bab93aa  podman      bridge
```

> By default, Podman creates a `bridge` network named `podman` that is used for container communication.

### Subtask 1.2: Inspect Default Network

You can inspect the default network using:

```bash
podman network inspect podman
```

This command will show detailed information such as:

* Subnet ranges
* Gateway IP
* DNS settings

This is especially useful when you're troubleshooting connectivity issues between containers or between host and container.

## Task 2: Inspect Network Settings

### Subtask 2.1: Create a Custom Bridge Network

To create a new bridge network for container isolation:

```bash
podman network create lab-network
```

Verify its creation:

```bash
podman network ls | grep lab-network
```

### Subtask 2.2: Inspect the Custom Network

Check details of the new custom network:

```bash
podman network inspect lab-network
```

You will see a JSON output including:

* Subnet configuration
* Gateway information
* IP Address Management (IPAM)

> **Tip:** If the command fails, check if Podman is running in rootless mode. Run `podman info` to verify.

## Task 3: Run Containers with Port Publishing

### Subtask 3.1: Run a Container with Port Mapping

Let’s run a basic Nginx container with host-to-container port mapping:

```bash
podman run -d --name webapp -p 8080:80 docker.io/library/nginx
```

* `-p 8080:80` maps host port 8080 to container port 80.
* `-d` runs the container in detached mode.

### Subtask 3.2: Verify Port Accessibility

Test if the container is accessible on port 8080:

```bash
curl http://localhost:8080
```

**Expected Output:** You should see the Nginx welcome page HTML.

>  **Troubleshooting:**
>
> * Ensure the container is running: `podman ps`
> * Check firewall rules: `sudo firewall-cmd --list-ports`

### Subtask 3.3: Attach Container to Custom Network

Let’s stop the running container and relaunch it with the custom network:

```bash
podman stop webapp

podman run -d --name webapp -p 8080:80 --network lab-network docker.io/library/nginx
```

To confirm that it's connected to the new network:

```bash
podman inspect webapp | grep -A 5 "Networks"
```
