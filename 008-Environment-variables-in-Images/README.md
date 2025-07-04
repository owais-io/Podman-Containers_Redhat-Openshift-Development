# Environment Variables in Images

## Task 1: Define Environment Variables in Containerfile

### Subtask 1.1: Create a Basic Containerfile

So let’s start by creating a new directory and moving into it:

```bash
mkdir env-lab && cd env-lab
```

Now we’ll create our `Containerfile` with some environment variables.

```bash
cat <<EOF > Containerfile
FROM registry.access.redhat.com/ubi8/ubi-minimal
ENV APP_NAME="MyApp" \\
    APP_VERSION="1.0" \\
    APP_ENV="development"
CMD echo "Running \$APP_NAME v\$APP_VERSION in \$APP_ENV mode"
EOF
```

### Subtask 1.2: Build and Run the Image

Now build the image:

```bash
podman build -t env-demo .
```

And run it:

```bash
podman run env-demo
```

**Expected Output:**

```
Running MyApp v1.0 in development mode
```

---

## Task 2: Override Environment Variables at Runtime

### Subtask 2.1: Override Using Command Line

We can override environment variables during container runtime using `-e`.

```bash
podman run -e APP_ENV="production" -e APP_VERSION="2.0" env-demo
```

**Expected Output:**

```
Running MyApp v2.0 in production mode
```

### Subtask 2.2: Use Environment Files

Another way is to use an environment file. Let's create one:

```bash
cat <<EOF > app.env
APP_NAME=ProductionApp
APP_VERSION=3.0
APP_ENV=staging
EOF
```

Now run the container with this file:

```bash
podman run --env-file=app.env env-demo
```

**Expected Output:**

```
Running ProductionApp v3.0 in staging mode
```

---

## Task 3: Inspect Variables in Running Containers

### Subtask 3.1: View Environment Variables

Let’s run the container in the background:

```bash
podman run -d --name env-container env-demo
```

Now inspect the environment variables:

```bash
podman exec env-container env
```

**Expected Output (partial):**

```
APP_NAME=MyApp
APP_VERSION=1.0
APP_ENV=development
```

---

## Subtask 3.2: Use ARG for Build-Time Variables

Sometimes, we need to pass values at build time instead of runtime. That’s where `ARG` comes in.

Let’s modify the `Containerfile`:

```bash
cat <<EOF > Containerfile
FROM registry.access.redhat.com/ubi8/ubi-minimal
ARG APP_BUILD_NUMBER
ENV APP_BUILD=\$APP_BUILD_NUMBER
CMD echo "Build number: \$APP_BUILD"
EOF
```

Now build the image using the build argument:

```bash
podman build --build-arg APP_BUILD_NUMBER=42 -t arg-demo .
```

And run it:

```bash
podman run arg-demo
```

**Expected Output:**

```
Build number: 42
```
