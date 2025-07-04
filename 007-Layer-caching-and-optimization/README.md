# Layer Caching and Optimization

## 🧪 Task 1: Create Initial Dockerfile

### 🎯 Subtask 1.1: Non-optimized Dockerfile

We start by writing a basic Dockerfile named `Dockerfile.initial`. This one is simple but not optimized.

```Dockerfile
FROM ubuntu:22.04
RUN apt-get update
RUN apt-get install -y curl wget
RUN apt-get install -y python3 python3-pip
RUN pip install flask
COPY app.py /app/
WORKDIR /app
CMD ["python3", "app.py"]
```

Also, we create a basic Flask app named `app.py`:

```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return "Hello from optimized container!"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)
```

### 🔧 Subtask 1.2: Build the Initial Image

Now we build the image using Podman:

```bash
podman build -t myapp:initial -f Dockerfile.initial .
```

And then check the image size:

```bash
podman images myapp:initial
```

📌 **Expected Output:** The image will be quite large — several hundred MB — and made up of many layers.

---

## 🚀 Task 2: Optimize the Dockerfile

### 🧹 Subtask 2.1: Layer Consolidation

Now let’s optimize our Dockerfile by combining the layers and doing some cleanup:

```Dockerfile
FROM ubuntu:22.04
RUN apt-get update && \
    apt-get install -y curl wget python3 python3-pip && \
    pip install flask && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
COPY app.py /app/
WORKDIR /app
CMD ["python3", "app.py"]
```

This file is named `Dockerfile.optimized`.

### 🔧 Subtask 2.2: Build the Optimized Image

Let’s build this optimized image:

```bash
podman build -t myapp:optimized -f Dockerfile.optimized .
```

Now compare both images:

```bash
podman images myapp:*
```

📌 **Expected Output:** The optimized image should be smaller in size than the initial one due to layer consolidation and cleanup.

---

## 🧠 Task 3: Understand Build Cache

### 🔁 Subtask 3.1: Cache Behavior

Now let’s play with the caching mechanism. First, make a small change in `app.py` (like adding a comment), then rebuild:

```bash
podman build -t myapp:optimized -f Dockerfile.optimized .
```

Observe which layers were reused from the cache and which ones were rebuilt.

### 💣 Subtask 3.2: Cache-Busting Techniques

To bust the cache intentionally, we add this in the Dockerfile:

```Dockerfile
ARG CACHEBUST=1
RUN echo "Cache bust: $CACHEBUST"
```

Then build it like this:

```bash
podman build -t myapp:optimized --build-arg CACHEBUST=$(date +%s) -f Dockerfile.optimized .
```

🛠️ **Tip:** If you want to completely ignore the cache:

```bash
podman build --no-cache -t myapp:optimized -f Dockerfile.optimized .
```

---

## 🔍 Task 4: Inspect Layers

### 🧱 Subtask 4.1: View Layer History

You can view the image history like this:

```bash
podman history myapp:optimized
podman inspect myapp:optimized
```

### 🔬 Subtask 4.2: Analyze with Dive

If you want to see detailed info layer-by-layer, install `dive`:

```bash
sudo apt-get install dive
```

Then run:

```bash
dive myapp:optimized
```

📌 **Note:** Dive will help you find which layers are taking up most space and if anything can be cleaned.

---

## 🏗️ Task 5: Advanced Optimization

### 🏁 Subtask 5.1: Multi-stage Builds

Now let’s go one step further and use a **multi-stage build** to separate build-time and runtime dependencies. This is what the `Dockerfile.multistage` looks like:

```Dockerfile
# Build stage
FROM ubuntu:22.04 as builder
RUN apt-get update && apt-get install -y python3 python3-pip
COPY requirements.txt .
RUN pip install -r requirements.txt

# Runtime stage
FROM ubuntu:22.04
COPY --from=builder /usr/local/lib/python3.10/dist-packages /usr/local/lib/python3.10/dist-packages
COPY app.py /app/
WORKDIR /app
CMD ["python3", "app.py"]
```

Make sure you create a `requirements.txt` with this line:

```
flask
```

Now build it:

```bash
podman build -t myapp:multistage -f Dockerfile.multistage .
```

📌 **Expected Result:** This final image will only contain the runtime components, making it much smaller and more secure.
