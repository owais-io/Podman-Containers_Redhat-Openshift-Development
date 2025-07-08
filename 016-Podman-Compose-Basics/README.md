# Podman Compose Basics

## Setup and Prerequisites

Before starting, ensure you have the following tools installed:

### Verify Podman
```bash
podman --version
````

> Expected: `podman version 3.x.x`

### Verify Podman Compose

```bash
podman-compose --version
```

> Expected: `podman-compose version x.x.x`

## Task 1: Create a Simple `podman-compose.yml` File

### Subtask 1.1: Understand the Structure

The `podman-compose.yml` file follows Docker Compose syntax (version 3+). It defines:

* **services**: your containers (like web, database, etc.)
* **volumes**: to persist data or mount code
* **ports**: to expose internal services externally
* **environment**: to configure container settings

### Subtask 1.2: Write Your Compose File

Create the project directory:

```bash
mkdir compose-lab && cd compose-lab
touch podman-compose.yml
```

Add the following content to `podman-compose.yml`:

```yaml
version: '3.8'

services:
  web:
    image: docker.io/nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html

  db:
    image: docker.io/postgres:13
    environment:
      POSTGRES_PASSWORD: example
```

## Task 2: Launch the Multi-Container App

### Subtask 2.1: Start the Application

```bash
podman-compose up -d
```

Expected messages:

* Network and volume creation
* Image pulling
* Container startup

### Subtask 2.2: Verify Container Status

```bash
podman ps
```

You should see both containers running:

```
CONTAINER ID  IMAGE                 ...   PORTS               NAMES
abc123        nginx:alpine         ...   0.0.0.0:8080->80/tcp  compose-lab_web_1
def456        postgres:13          ...   5432/tcp             compose-lab_db_1
```

## Task 3: Test the Application

### Subtask 3.1: Access Web Service

Set up a sample HTML page:

```bash
mkdir html
echo "<h1>Hello from Podman Compose!</h1>" > html/index.html
```

Now visit: [http://localhost:8080](http://localhost:8080)
Or check via `curl`:

```bash
curl http://localhost:8080
```

Expected output:

```
<h1>Hello from Podman Compose!</h1>
```

### Subtask 3.2: Access Database Service

Enter the Postgres container:

```bash
podman exec -it compose-lab_db_1 psql -U postgres
```

Run a test query:

```sql
SELECT version();
\q
```

You should see a PostgreSQL version string like:

```
PostgreSQL 13.x on x86_64-pc-linux-gnu...
```

## Task 4: Stop and Clean Up

### Subtask 4.1: Bring Down the App

```bash
podman-compose down
```

Expected output:

* Stopping and removing containers
* Removing the network

### Subtask 4.2: Ensure Cleanup

Check that no containers are left:

```bash
podman ps -a
```

> Output should be empty.

## Troubleshooting Tips

* **Port already in use?**
  Change `8080:80` to something like `8081:80`.

* **SELinux issues on volumes?**
  Use the `:Z` flag:

  ```yaml
  volumes:
    - ./html:/usr/share/nginx/html:Z
  ```

* **Image not found or failed to pull?**
  Make sure you are connected to the internet and using the correct image name.

## Notes

* This lab uses only officially available images from Docker Hub.
* `podman-compose` behaves like `docker-compose`, but works with Podman under the hood.
* Files and configurations are kept minimal for clarity and ease of use.
