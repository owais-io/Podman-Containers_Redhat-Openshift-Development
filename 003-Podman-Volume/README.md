# Task 3: Volume Mounts with Podman

## Introduction

Here, we will learn how to mount host directories into containers using Podman. Volume mounting is one of the most powerful features of containerization technology. It allows you to share files and directories between your host machine and the running container, which can be very useful for development, testing, and persistent data storage.

So, this task will help you understand how to mount a directory from your host system into a container and serve its content using an Nginx web server.

---

## Creating a Host Directory

First of all, we will create a new directory on our host system that will contain a sample HTML file. This file will later be served through the Nginx container.

```bash
mkdir ~/nginx-content
echo "Hello from host!" > ~/nginx-content/index.html
````

In this step, we are using the `mkdir` command to create a new folder and the `echo` command to create a simple HTML file inside it.

---

## Running the Nginx Container with Volume Mount

Now that the directory is created, we will start an Nginx container using Podman. While running the container, we will mount the previously created directory into the container’s default web server path.

```bash
podman run -d -p 8081:80 -v ~/nginx-content:/usr/share/nginx/html docker.io/library/nginx:alpine
```

Here:

* `-d` runs the container in detached mode.
* `-p 8081:80` maps port 8081 of your host to port 80 inside the container.
* `-v ~/nginx-content:/usr/share/nginx/html` mounts your host directory into the container.
* `nginx:alpine` is the lightweight version of the official Nginx image.

---

## Verifying the Mounted Content

Once the container is up and running, you can check if the mounted content is being served by Nginx by using the `curl` command:

```bash
curl http://localhost:8081
```

You should see the following output:

```
Hello from host!
```

This confirms that the volume was successfully mounted and the Nginx server is serving content directly from your host system.

---

## Conclusion

Through this task, you have learned how to:

* Create a directory on your host system.
* Add a file to it.
* Mount that directory into a running container using Podman.
* Serve the content from that directory using the Nginx web server.

This is a foundational step for many real-world DevOps workflows, where containers need access to external configuration files, logs, or content directories.

```
