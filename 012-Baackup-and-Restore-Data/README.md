# Backup and Restore Data

##  Setup

First, verify that Podman is installed on your system:

```bash
podman --version
````

Create your working directory:

```bash
mkdir -p ~/backup-lab && cd ~/backup-lab
```

##  Task 1: Perform Database Dumps Inside Container

###  Subtask 1.1: Create a MySQL Container

Run the following command to launch a containerized MySQL server:

```bash
podman run -d --name mysql-db \
  -e MYSQL_ROOT_PASSWORD=redhat \
  -e MYSQL_DATABASE=testdb \
  -e MYSQL_USER=testuser \
  -e MYSQL_PASSWORD=testpass \
  docker.io/library/mysql:8.0
```

Verify the container is running:

```bash
podman ps -f name=mysql-db
```

###  Subtask 1.2: Create Sample Data

Access the MySQL container:

```bash
podman exec -it mysql-db mysql -u root -predhat
```

Create a sample table and insert data:

```sql
USE testdb;
CREATE TABLE employees (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(50));
INSERT INTO employees (name) VALUES ('John Doe'), ('Jane Smith');
SELECT * FROM employees;
```

Exit MySQL:

```sql
exit
```

###  Subtask 1.3: Create Database Dump

Run the dump command from the container:

```bash
podman exec mysql-db /usr/bin/mysqldump -u root -predhat testdb > testdb_dump.sql
```

Verify the dump:

```bash
ls -l testdb_dump.sql
head -n 5 testdb_dump.sql
```
##  Task 2: Store Dumps on Volumes

###  Subtask 2.1: Create Persistent Volume

```bash
podman volume create backup-vol
```

###  Subtask 2.2: Copy Dump to Volume

Create and enter a temporary container:

```bash
podman run -it --rm -v backup-vol:/backup alpine sh
```

Inside the container:

```sh
mkdir -p /backup/mysql
exit
```

Copy the dump file into the volume:

```bash
podman cp testdb_dump.sql $(podman create --name temp -v backup-vol:/backup alpine):/backup/mysql/
podman rm temp
```

Verify the contents of the volume:

```bash
podman run --rm -v backup-vol:/backup alpine ls -l /backup/mysql
```

##  Task 3: Restore Data from Dumps

###  Subtask 3.1: Create New MySQL Container

```bash
podman run -d --name mysql-restore \
  -e MYSQL_ROOT_PASSWORD=redhat \
  -e MYSQL_DATABASE=testdb \
  -e MYSQL_USER=testuser \
  -e MYSQL_PASSWORD=testpass \
  docker.io/library/mysql:8.0
```

###  Subtask 3.2: Restore Data

Copy the dump file from the volume:

```bash
podman run --rm -v backup-vol:/backup alpine cp /backup/mysql/testdb_dump.sql /tmp/
podman cp $(podman create --name temp -v backup-vol:/backup alpine):/tmp/testdb_dump.sql .
podman rm temp
```

Restore the data:

```bash
podman exec -i mysql-restore mysql -u root -predhat testdb < testdb_dump.sql
```

###  Subtask 3.3: Verify Restoration

Check if data was restored successfully:

```bash
podman exec -it mysql-restore mysql -u root -predhat -e "SELECT * FROM testdb.employees;"
```

Expected output:

```
+----+------------+
| id | name       |
+----+------------+
|  1 | John Doe   |
|  2 | Jane Smith |
+----+------------+
```

---

## Troubleshooting Tips

* **MySQL container not starting?**

  ```bash
  podman logs mysql-db
  ```

* **Permission issues with volumes?**
  Use SELinux label:

  ```bash
  -v backup-vol:/backup:Z
  ```

* **Dump not generated?**
  Ensure MySQL credentials and database name are correct.

##  Cleanup

To clean up all resources created during the lab:

```bash
podman stop mysql-db mysql-restore
podman rm mysql-db mysql-restore
podman volume rm backup-vol
rm -f testdb_dump.sql
```

