# Exploring Podman CLI

## 📋 Task 1: Listing Containers

The very first step is to understand how to **list your containers** — whether they’re running or stopped.

###  Step 1: List Running Containers
To view currently active containers, run:
```bash
podman ps
````

If there are no running containers, you’ll get an empty output.

###  Step 2: List All Containers (Running + Stopped)

```bash
podman ps -a
```

This will show you all containers — running, exited, or created but not started.

>  **Troubleshooting Tip**:
> If you see permission errors, try using `sudo` or set up **rootless Podman** properly.


##  Task 2: Running a Container

Now let’s run our first container using **Alpine Linux**, a lightweight image often used for testing.

###  Step 1: Start Alpine Container

```bash
podman run -it --name my_alpine alpine sh
```

#### Explanation:

* `-it`: Interactive mode with terminal
* `--name my_alpine`: Assigns the name `my_alpine` to the container
* `alpine`: The image used
* `sh`: Shell to run inside

This will drop you into an interactive shell inside the container.
To exit the container, simply type:

```bash
exit
```

###  Step 2: Confirm It Ran

```bash
podman ps
```

You should see `my_alpine` listed as a running container.

##  Task 3: Stopping a Container

Let’s stop the container we just started.

###  Step 1: Stop It

```bash
podman stop my_alpine
```

###  Step 2: Confirm the Status

```bash
podman ps -a
```

You should now see the container listed with **Exited** status.


##  Task 4: Restarting a Container

You can bring back a stopped container easily.

###  Step 1: Restart

```bash
podman restart my_alpine
```

###  Step 2: Confirm It's Running Again

```bash
podman ps
```


##  Task 5: Removing a Container

Once you're done with a container, you can remove it to free up resources.

###  Step 1: Stop (if it's still running)

```bash
podman stop my_alpine
```

###  Step 2: Remove

```bash
podman rm my_alpine
```

###  Step 3: Confirm Deletion

```bash
podman ps -a
```

You should no longer see `my_alpine` listed.


## 🔍 Task 6: Inspecting Container Details

This is where you get to peek under the hood of your container.

###  Step 1: Run a Detached NGINX Container

```bash
podman run -d --name nginx_container nginx
```

* `-d`: Detached mode, runs in the background

###  Step 2: Inspect the Container

```bash
podman inspect nginx_container
```

This returns a **detailed JSON** output including config, networking, mounts, and more.

###  Step 3: Clean Up

```bash
podman stop nginx_container && podman rm nginx_container
```

