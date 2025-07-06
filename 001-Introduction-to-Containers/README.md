# Podman Basics: Hands-on Lab Guide

## Setup Instructions

Let's begin by installing Podman based on your Linux distribution.

### For Ubuntu/Debian
```bash
sudo apt-get update
sudo apt-get install -y podman
````

### For Fedora/RHEL/CentOS

```bash
sudo dnf install -y podman
```

### Verify Installation

```bash
podman --version
```

Expected output should look like:

```
podman version 4.x.x
```

##  Lab Tasks

### Task 1: Understanding Container Basics

Containers are lightweight and portable executable packages that carry everything needed to run an application — including the code, libraries, dependencies, and configuration.

Some key characteristics of containers are:

* **Isolation**: Each container runs independently of others and of the host.
* **Portability**: Runs consistently across development, testing, and production.

### Task 2: Running a Simple Container with Podman

This section will walk you through running your first container.

#### Step 1: Pull an Image

```bash
podman pull hello-world
```

Expected output:

```
Trying to pull docker.io/library/hello-world:latest...
Status: Downloaded newer image for hello-world:latest
```

#### Step 2: Run the Container

```bash
podman run hello-world
```

Expected output includes:

```
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

#### Step 3: Check Container Status

```bash
podman ps -a
```

You should see something like:

```
CONTAINER ID  IMAGE                           COMMAND     CREATED         STATUS                     PORTS       NAMES
abc123        docker.io/library/hello-world   /hello      5 seconds ago   Exited (0) 2 seconds ago               laughing_wright
```

### Task 3: Exploring Container Isolation

#### Step 1: Launch an Interactive Shell

We’ll run Alpine Linux in interactive mode.

```bash
podman run -it alpine sh
```

You should see:

```
/ #
```

#### Step 2: Verify Container OS

Inside the container shell, run:

```bash
cat /etc/os-release
```

This will confirm you are inside an Alpine Linux container. To exit:

```bash
exit
```

---

##  Troubleshooting Tips

* **Permission Issues**: Use `sudo` if required:

  ```bash
  sudo podman run hello-world
  ```

* **Rootless Mode** (recommended):

  ```bash
  podman system migrate
  ```

* **Networking Issues**: Ensure your system is online. You can also run with detailed logs:

  ```bash
  podman --log-level=debug pull hello-world
  ```
