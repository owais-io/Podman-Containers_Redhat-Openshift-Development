# Compose Dependencies with Docker


## Setup

Before we begin, create a new working directory for this lab and move into it:

```bash
mkdir compose-dependencies-lab
cd compose-dependencies-lab
````

## Task 1: Using `depends_on` to Define Service Dependencies

In this task, we’ll create a simple `docker-compose.yml` file and explore how `depends_on` helps us define service startup order.

### Step 1.1: Create a Basic Compose File

Create a file named `docker-compose.yml` and add the following content:

```yaml
version: '3.8'

services:
  redis:
    image: redis:alpine
    ports:
      - "6379:6379"
  
  webapp:
    image: nginx:alpine
    ports:
      - "8080:80"
    depends_on:
      - redis
```

### Step 1.2: Understanding `depends_on`

The `depends_on` directive ensures that Redis starts before the `webapp` service.
However, **note** that this does **not** guarantee that Redis is ready to accept connections — it only ensures the container starts before the dependent one.

### Step 1.3: Start the Services

```bash
docker-compose up -d
```

You can verify if the services are running using:

```bash
docker-compose ps
```

## Task 2: Scaling Service Replicas

This task demonstrates how to run multiple replicas of a single service using the `--scale` flag.

### Step 2.1: Modify the Compose File

Update your `docker-compose.yml` file to look like this:

```yaml
version: '3.8'

services:
  redis:
    image: redis:alpine
    ports:
      - "6379:6379"
  
  webapp:
    image: nginx:alpine
    ports:
      - "8080-8082:80"
    depends_on:
      - redis
    deploy:
      replicas: 3
```

### Step 2.2: Start Scaled Services

```bash
docker-compose up -d --scale webapp=3
```

### Step 2.3: Verify Scaling

Use the following commands:

```bash
docker-compose ps
docker-compose exec webapp hostname
```

You should see:

* One Redis container running.
* Three webapp containers running with different hostnames.

## Task 3: Test Inter-Service Communication

In this task, we’ll connect a custom Flask application to Redis and verify that they communicate correctly.

### Step 3.1: Create the Test App

Create a file named `app.py` with the following content:

```python
from flask import Flask
import redis
import os

app = Flask(__name__)
redis_host = os.environ.get('REDIS_HOST', 'redis')
redis_client = redis.Redis(host=redis_host, port=6379)

@app.route('/')
def hello():
    count = redis_client.incr('hits')
    return f'Hello World! This page has been viewed {count} times.\n'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

Now, create a `Dockerfile` to containerize the app:

```Dockerfile
FROM python:3.9-alpine
WORKDIR /app
COPY . .
RUN pip install flask redis
CMD ["python", "app.py"]
```

### Step 3.2: Update the Compose File

Update your `docker-compose.yml` again:

```yaml
version: '3.8'

services:
  redis:
    image: redis:alpine
  
  webapp:
    build: .
    ports:
      - "5000:5000"
    environment:
      - REDIS_HOST=redis
    depends_on:
      - redis
```

### Step 3.3: Build and Test

Build and run the containers:

```bash
docker-compose up -d --build
```

Test the Flask application:

```bash
curl http://localhost:5000
```

You should see output like:

```
Hello World! This page has been viewed 1 times.
```

Refreshing or re-running the `curl` command should increment the counter.

## Troubleshooting Tips

Here are a few common issues you might encounter and how to resolve them:

* **Services fail to start?**

  * Check logs with `docker-compose logs`
  * Ensure no port conflicts
  * Confirm Docker has enough memory and CPU

* **Scaling doesn't work?**

  * Use Docker Compose v2+
  * Avoid fixed ports if running multiple replicas

* **Connectivity issues?**

  * Double-check service names in your code/environment
  * Use healthchecks if a service needs to wait for another to be fully ready
