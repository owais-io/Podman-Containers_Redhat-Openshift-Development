# ENTRYPOINT and CMD Lab with Podman

## Task 1: Writing a Containerfile with ENTRYPOINT and CMD

### Subtask 1.1: Create a Basic Containerfile

Create a file named `Containerfile` with the following content:

```Dockerfile
# Base image
FROM registry.access.redhat.com/ubi9/ubi-minimal

# Set ENTRYPOINT to a shell script
ENTRYPOINT ["echo", "Entrypoint says:"]

# Set default CMD arguments
CMD ["Default CMD message"]
```

#### Explanation:

* `ENTRYPOINT` defines the main executable to run inside the container.
* `CMD` provides default arguments passed to the ENTRYPOINT.
* Together, this results in:

  ```sh
  echo "Entrypoint says:" "Default CMD message"
  ```

---

### Subtask 1.2: Build the Image

Use `podman` to build the container image:

```sh
podman build -t entrypoint-demo .
```

Expected build output:

```
STEP 1/3: FROM registry.access.redhat.com/ubi9/ubi-minimal
STEP 2/3: ENTRYPOINT ["echo", "Entrypoint says:"]
STEP 3/3: CMD ["Default CMD message"]
COMMIT entrypoint-demo
```

---

## Task 2: Testing Container Behavior

### Subtask 2.1: Run with Default ENTRYPOINT and CMD

```sh
podman run --rm entrypoint-demo
```

Expected output:

```
Entrypoint says: Default CMD message
```

### Subtask 2.2: Override CMD at Runtime

```sh
podman run --rm entrypoint-demo "Custom message"
```

Expected output:

```
Entrypoint says: Custom message
```

> When you provide arguments at runtime, they override the `CMD` only—not the `ENTRYPOINT`.

---

## Task 3: Advanced ENTRYPOINT/CMD Combinations

### Subtask 3.1: Create a Script-based ENTRYPOINT

Create a file called `greet.sh`:

```sh
#!/bin/sh
echo "Welcome to $1 from $2"
```

Make it executable:

```sh
chmod +x greet.sh
```

Update your `Containerfile` like so:

```Dockerfile
FROM registry.access.redhat.com/ubi9/ubi-minimal

COPY greet.sh /usr/local/bin/
ENTRYPOINT ["/usr/local/bin/greet.sh"]
CMD ["OpenShift Lab", "Red Hat"]
```

Build the image:

```sh
podman build -t greet-demo .
```

---

### Subtask 3.2: Test Script-based Container

Run with default values:

```sh
podman run --rm greet-demo
```

Expected output:

```
Welcome to OpenShift Lab from Red Hat
```

Override both arguments:

```sh
podman run --rm greet-demo "Container Workshop" "Instructor"
```

Expected output:

```
Welcome to Container Workshop from Instructor
```

---

## Task 4: Overriding ENTRYPOINT

### Subtask 4.1: Use `--entrypoint` to Replace ENTRYPOINT

```sh
podman run --rm --entrypoint echo greet-demo "This completely replaces the ENTRYPOINT"
```

Expected output:

```
This completely replaces the ENTRYPOINT
```

> **Note**: If you face permission issues with the script, double-check file permissions on both the host and inside the image.

---

## Task 5: Shell vs Exec Form

###  Subtask 5.1: Shell-form ENTRYPOINT Example

Update the `Containerfile` to use shell form:

```Dockerfile
FROM registry.access.redhat.com/ubi9/ubi-minimal

# Shell form ENTRYPOINT (runs via /bin/sh -c)
ENTRYPOINT echo "Shell form ENTRYPOINT:"
CMD echo "Shell form CMD"
```

###  Key Difference

| Feature     | Exec Form                   | Shell Form                       |
| ----------- | --------------------------- | -------------------------------- |
| Behavior    | Executes directly           | Runs via `/bin/sh -c`            |
| Environment | More secure and predictable | May involve shell variable eval  |
| Recommended | Yes                         | Not for most production cases    |

---
