# Troubleshooting Containers

##  Setup Instructions

To get started, first install Podman (if it's not already installed).

For **RHEL, CentOS, Fedora**:
```bash
sudo dnf install -y podman
````

For **Debian/Ubuntu**:

```bash
sudo apt-get install -y podman
```

Then pull a sample container image:

```bash
podman pull docker.io/library/nginx:alpine
```

Run a test container:

```bash
podman run -d --name nginx-test -p 8080:80 nginx:alpine
```

##  Task 1: Viewing Logs with Filters

### 1.1: View Basic Logs

To view logs from the container:

```bash
podman logs nginx-test
```

To stream logs in real-time:

```bash
podman logs -f nginx-test
```

> Press `Ctrl+C` to stop real-time streaming.

### 1.2: Filter Logs by Time

To show logs from the last 5 minutes:

```bash
podman logs --since 5m nginx-test
```

To view only the last 10 lines of logs:

```bash
podman logs --tail 10 nginx-test
```

> **Tip:** If no logs appear, make sure the container is running:

```bash
podman ps
```

##  Task 2: Inspecting Container State

### 2.1: Check Container Status

To list currently running containers:

```bash
podman ps
```

To get full inspection data for a container:

```bash
podman inspect nginx-test
```

Key fields to observe:

* `"State"`: Check if the container is running or exited
* `"ExitCode"` and `"Error"`: Helps understand failure reasons
* `"Config"`: See entrypoint and environment variables
* `"NetworkSettings"`: Check IP address and ports

### 2.2: Monitor Resource Usage

To get live statistics like CPU, memory, and network:

```bash
podman stats nginx-test
```

> **Tip:** If the container hangs or misbehaves, stop and restart it:

```bash
podman stop nginx-test
podman start nginx-test
```

##  Task 3: Executing Commands Inside the Container

### 3.1: Access the Container Shell

To get an interactive shell:

```bash
podman exec -it nginx-test /bin/sh
```

Inside the shell, you can check running processes:

```bash
ps aux
```

### 3.2: Debugging Nginx from Inside

To view the Nginx configuration:

```bash
cat /etc/nginx/nginx.conf
```

To test if the service is working inside:

```bash
curl localhost
```

> If `/bin/sh` doesn’t work, try `/bin/bash` (if available in the container image).

##  Lab Completion Checklist

* [x] Viewed container logs using basic and advanced filters
* [x] Inspected container status, configuration, and network settings
* [x] Checked resource usage with `podman stats`
* [x] Used `podman exec` to enter container and debug the Nginx setup

##  Cleanup

Once you're done with the lab:

```bash
podman stop nginx-test
podman rm nginx-test
```
