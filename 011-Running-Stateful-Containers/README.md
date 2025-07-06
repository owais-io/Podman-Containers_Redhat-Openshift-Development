# Podman-Based Persistent MySQL and PostgreSQL Containers

This repository guides you through running MySQL and PostgreSQL containers using Podman with persistent storage volumes. It helps you understand how to launch containers, configure them using environment variables, test database operations, and verify data persistence after container removal.

## Task 1: Running MySQL Container with Persistent Storage

This section explains how to configure a MySQL container that retains its data even after being stopped and removed.

### Subtask 1.1: Create Persistent Volume

To begin, create a local directory that will serve as the persistent storage location for MySQL data.

```bash
mkdir -p mysql-data
````

This folder will be mounted inside the container and will store all the database files.

### Subtask 1.2: Launch MySQL Container

Run the following command to launch a MySQL 8.0 container with persistent volume and preconfigured environment variables:

```bash
podman run -d \
  --name mysql-db \
  -e MYSQL_ROOT_PASSWORD=redhat123 \
  -e MYSQL_DATABASE=testdb \
  -e MYSQL_USER=testuser \
  -e MYSQL_PASSWORD=user123 \
  -v $(pwd)/mysql-data:/var/lib/mysql \
  -p 3306:3306 \
  docker.io/library/mysql:8.0
```

**Explanation of Key Flags:**

* `-v`: Mounts the host directory as a volume in the container.
* `-e`: Sets environment variables for the database root password, user, and database.
* `-p`: Maps the container port 3306 to the host port 3306.

To verify that the container is running:

```bash
podman ps -a --format "table {{.ID}}\t{{.Names}}\t{{.Status}}"
```

---

### Subtask 1.3: Test Database Connection

Connect to the MySQL shell inside the container:

```bash
podman exec -it mysql-db mysql -u testuser -puser123 testdb
```

Once inside the MySQL shell, run the following commands:

```sql
CREATE TABLE lab_data (id INT AUTO_INCREMENT PRIMARY KEY, message VARCHAR(255));
INSERT INTO lab_data (message) VALUES ('Persistent test data');
SELECT * FROM lab_data;
exit;
```

You should see the inserted row displayed.

## Task 2: Verifying Data Persistence

Now let's verify whether the data survives a container removal and re-creation.

### Subtask 2.1: Remove Existing Container

Stop and remove the container:

```bash
podman stop mysql-db
podman rm mysql-db
```

### Subtask 2.2: Recreate Container with Same Volume

Now re-run the container with only the root password and volume mounted:

```bash
podman run -d \
  --name mysql-db \
  -e MYSQL_ROOT_PASSWORD=redhat123 \
  -v $(pwd)/mysql-data:/var/lib/mysql \
  -p 3306:3306 \
  docker.io/library/mysql:8.0
```

### Subtask 2.3: Verify Data Survival

Check if the previously created table and data are still there:

```bash
podman exec -it mysql-db mysql -u testuser -puser123 testdb -e "SELECT * FROM lab_data;"
```

You should see the `lab_data` table and the previously inserted record.

## Task 3: PostgreSQL Implementation (Optional Challenge)

This optional section sets up a PostgreSQL container with persistence.

### Subtask 3.1: Create Volume Directory

```bash
mkdir -p pg-data
```

### Subtask 3.2: Launch PostgreSQL Container

```bash
podman run -d \
  --name postgres-db \
  -e POSTGRES_PASSWORD=redhat123 \
  -e POSTGRES_USER=testuser \
  -e POSTGRES_DB=testdb \
  -v $(pwd)/pg-data:/var/lib/postgresql/data \
  -p 5432:5432 \
  docker.io/library/postgres:13
```

### Subtask 3.3: Insert Data and Verify

Run the following to connect and insert test data:

```bash
podman exec -it postgres-db psql -U testuser -d testdb -c "CREATE TABLE lab_data (id SERIAL PRIMARY KEY, message TEXT); INSERT INTO lab_data (message) VALUES ('Postgres persistent data');"
```

Repeat the stop/remove and recreate steps as done in Task 2, and verify persistence using:

```bash
podman exec -it postgres-db psql -U testuser -d testdb -c "SELECT * FROM lab_data;"
```

## Troubleshooting Tips

### Permission Issues

Sometimes permission errors may arise due to UID mismatch. You can fix them using:

```bash
sudo chown -R 1001:1001 mysql-data/
```

### Port Conflicts

Ensure the desired port (like 3306 for MySQL) is not already in use:

```bash
ss -tulnp | grep 3306
```

### Viewing Container Logs

If the container fails to start or behaves unexpectedly, check logs:

```bash
podman logs mysql-db
```

