
---

````markdown
# Creating a Simple Custom Nginx Container Using Podman

This repo is all about, how to create a basic container image using Podman. We’ll build a custom Nginx-based image that serves a personalized welcome page. So let’s get started!

---

## Step 1: Create a Project Directory

First things first, create a new directory for your project and move into it.

```bash
mkdir custom-nginx && cd custom-nginx
````

This directory will contain everything related to your custom container image.

---

## Step 2: Create a Custom HTML File

Next, we’ll create a simple `index.html` file with some welcome text.

```bash
echo "<h1>Welcome to My Custom Nginx Container!</h1>" > index.html
```

This file will be served as the default page by our Nginx server.

---

## Step 3: Write the Containerfile

Now comes the important part — writing the **Containerfile** (same as Dockerfile; Podman supports both). Create a file named `Containerfile` and add the following content to it:

```Dockerfile
# Use the official Nginx base image
FROM docker.io/nginx:alpine

# Set an environment variable
ENV AUTHOR="OpenShift Developer"

# Copy the custom HTML file to the Nginx web root
COPY index.html /usr/share/nginx/html

# Run a command to print a message (for demonstration)
RUN echo "Container built by $AUTHOR" > /build-info.txt
```

Let me break this down for you:

* `FROM`: This tells Podman to use the Alpine-based Nginx image as the starting point.
* `ENV`: This sets an environment variable named AUTHOR.
* `COPY`: This moves the custom HTML file from your local directory into the container's web directory.
* `RUN`: This runs a simple command while building the image (just for demo purposes).

---

## Step 4: Build and Tag the Image

Once you’ve written your Containerfile, it’s time to build your image using Podman.

```bash
podman build -t my-custom-nginx .
```

* The `-t` flag assigns a name or tag to the image.
* The dot (`.`) refers to the current directory as the build context.

### Expected Output:

```text
STEP 1/4: FROM docker.io/nginx:alpine
STEP 2/4: ENV AUTHOR="OpenShift Developer"
STEP 3/4: COPY index.html /usr/share/nginx/html
STEP 4/4: RUN echo "Container built by $AUTHOR" > /build-info.txt
COMMIT my-custom-nginx
```

---

## Step 5: Verify the Image

Let’s check if our image was built successfully:

```bash
podman images
```

### Expected Output:

```text
REPOSITORY               TAG      IMAGE ID       CREATED          SIZE
localhost/my-custom-nginx  latest   xxxxxxxxxxxx   1 minute ago    25.4 MB
```

You should see your image listed here.

---

## Step 6: Run the Container

Now let’s run a container from the image we just created:

```bash
podman run -d -p 8080:80 my-custom-nginx
```

* The `-d` flag runs the container in detached mode (in the background).
* `-p 8080:80` maps port 8080 on your host to port 80 in the container.

---

## Step 7: Verify the Running Container

To check if your container is up and running, use:

```bash
podman ps
```

This will list all the active containers.

---

## Step 8: Test the Web Server

Now it’s time for the fun part. Open your web browser and go to:

```
http://localhost:8080
```

You should see your custom welcome page that says:

```html
Welcome to My Custom Nginx Container!
```

---

That’s it for today! 🎉
With just a few steps, you’ve successfully created and deployed a custom Nginx container using Podman.

Thanks!

