# Podman Volume and Bind Mount Lab

This README walks you through how to manage persistent data in Podman using **named volumes** and **bind mounts**. You will learn how to create, inspect, and use volumes in containers, and how to verify that data persists even after the container is removed. You’ll also explore how to link host directories directly into containers using bind mounts.


## Task 1: Creating Named Volumes

### Subtask 1.1 – Creating a Named Volume

To create a named volume that is managed by Podman and persists even if the container is deleted, run:

```bash
podman volume create myapp_data
````

You should see:

```
myapp_data
```

#### Verify the volume

```bash
podman volume ls
```

Look for `myapp_data` in the list.

### Subtask 1.2 – Inspect the Volume

To inspect the volume and see where its data is stored on the host:

```bash
podman volume inspect myapp_data
```

Sample output:

```json
[
    {
        "Name": "myapp_data",
        "Driver": "local",
        "Mountpoint": "/var/lib/containers/storage/volumes/myapp_data/_data",
        "CreatedAt": "2023-10-05T12:00:00Z",
        "Labels": {},
        "Scope": "local"
    }
]
```

 **Note**: The `Mountpoint` is the actual location where your container data is saved on the host.

## Task 2: Mounting Volumes in Containers

### Subtask 2.1 – Running a Container with a Volume

We can attach a volume to a container like this:

```bash
podman run -d --name webapp -v myapp_data:/var/www/html docker.io/library/nginx
```

What this does:

* `-v myapp_data:/var/www/html` mounts the named volume inside the container at `/var/www/html`.

To confirm it’s mounted (and currently empty):

```bash
podman exec webapp ls /var/www/html
```

*No output is expected at this point.*

### Subtask 2.2 – Persisting Data into the Volume

Let’s write some data into the mounted volume:

```bash
podman exec webapp sh -c "echo 'Hello, Volume!' > /var/www/html/index.html"
```

Now confirm it was saved:

```bash
podman exec webapp cat /var/www/html/index.html
```

Expected output:

```
Hello, Volume!
```

### Subtask 2.3 – Verifying Persistence Across Containers

First, remove the container:

```bash
podman rm -f webapp
```

Then create a new container using the same volume:

```bash
podman run -d --name webapp_new -v myapp_data:/var/www/html docker.io/library/nginx
```

Check the file again:

```bash
podman exec webapp_new cat /var/www/html/index.html
```

You should see:

```
Hello, Volume!
```

 **Key Takeaway**: Because the file was stored in a named volume, it persisted across containers.

## Task 3: Using Bind Mounts with Host Directories

Bind mounts allow you to connect a **host directory** directly into a container.

### Subtask 3.1 – Creating a Host Directory

Create a new directory on the host and add a file:

```bash
mkdir ~/host_data
echo "Hello, Bind Mount!" > ~/host_data/index.html
```

### Subtask 3.2 – Running a Container with a Bind Mount

Now run a container that binds the host directory:

```bash
podman run -d --name bind_example -v ~/host_data:/usr/share/nginx/html:Z docker.io/library/nginx
```

Here:

* `-v ~/host_data:/usr/share/nginx/html` links the host folder to the container path.
* `:Z` is added for proper SELinux context labeling (important for systems with SELinux enabled).

Verify:

```bash
podman exec bind_example cat /usr/share/nginx/html/index.html
```

Expected output:

```
Hello, Bind Mount!
```

### Subtask 3.3 – Modifying Data on the Host and Verifying in Container

Append some more data on the host:

```bash
echo "Updated content!" >> ~/host_data/index.html
```

Now read the file again inside the container:

```bash
podman exec bind_example cat /usr/share/nginx/html/index.html
```

Expected output:

```
Hello, Bind Mount!
Updated content!
```

 **Key Takeaway**: Changes on the host are reflected immediately in the container when using bind mounts.

## Troubleshooting Tips

* **Permission Denied?** Use `:Z` or `:z` when mounting bind volumes on SELinux-enabled systems.
* **Volume Not Found?** Double-check using `podman volume ls`.
* **Path Issues?** Always use **absolute paths** when specifying host directories.

