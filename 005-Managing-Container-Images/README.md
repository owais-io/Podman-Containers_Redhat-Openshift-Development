# Managing Container Images

##  Setup Instructions

Before anything else, ensure Podman is installed on your system. If it isn’t installed already, run the following:

```bash
sudo apt-get update
sudo apt-get install -y podman
````

Once installed, verify it with:

```bash
podman --version
```


##  Task 1: Search for Images on Docker Hub

Searching for container images is often your first step before deployment. You can use the `podman search` command to do this.

### Subtask 1.1: Search for an Image

```bash
podman search ubuntu
```

To narrow it down to official images only:

```bash
podman search --filter=is-official=true ubuntu
```

This will display results with columns such as `NAME`, `DESCRIPTION`, `STARS`, and whether it’s official.

> **Note:** If you're rate-limited while searching Docker Hub, you can log in using:
>
> ```bash
> podman login docker.io
> ```

##  Task 2: Pull Container Images

Pulling an image downloads it to your local system so it can be used to launch containers.

### Subtask 2.1: Pull the Latest Ubuntu Image

```bash
podman pull docker.io/library/ubuntu:latest
```

You can verify it by listing local images:

```bash
podman images
```

You should see the Ubuntu image in the list.

### Subtask 2.2: Pull an Older Version

```bash
podman pull docker.io/library/ubuntu:20.04
```

Again, check the image list:

```bash
podman images
```

> **Tip:** Tags (like `latest`, `20.04`, etc.) help you control and manage different versions of the same base image.

##  Task 3: Inspect Image Metadata

Podman allows you to inspect detailed metadata about an image. This is useful for understanding architecture, environment variables, labels, and more.

### Subtask 3.1: View Basic Information

```bash
podman inspect docker.io/library/ubuntu:latest
```

This will return a JSON output containing all the metadata.

### Subtask 3.2: Extract Specific Info (e.g. Environment Variables)

```bash
podman inspect --format "{{.Config.Env}}" docker.io/library/ubuntu:latest
```

> **Concept:** Image inspection gives visibility into how a container might behave once it is started.

##  Task 4: Remove Container Images

Cleaning up unused or outdated images is an important maintenance task.

### Subtask 4.1: Remove a Specific Image

To remove a specific version (e.g., Ubuntu 20.04):

```bash
podman rmi docker.io/library/ubuntu:20.04
```

Check again:

```bash
podman images
```

### Subtask 4.2: Remove All Unused Images

```bash
podman image prune -a
```

> **Note:** If an image is still being used by a container, you’ll need to force remove it using its `IMAGE_ID`:

```bash
podman rmi -f IMAGE_ID
```
