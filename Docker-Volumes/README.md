# Docker Volumes — Persistent Storage the Easy Way

In the last parts, we built images and ran containers. In this article, we’ll solve a critical problem: **how to keep your data when containers are stopped, removed, or restarted**. Let’s discuss Docker volumes.

---

## What You’ll Learn

* Why container files disappear and how volumes fix it
* The 3 storage types: **named volumes**, **bind mounts**, and **tmpfs**
* `-v` vs `--mount` syntax (and when to use which)
* Real demos:

  * MySQL without a volume → data loss
  * MySQL with a named volume → data persists
  * Nginx bind mount → live-edit files from your host
* Managing volumes: create, list, inspect, backup/restore, remove
* Common pitfalls: permissions, Windows paths, SELinux, anonymous volumes
* Docker Compose equivalents (preview for Part 6)

---

## The Problem: Containers Are Ephemeral

A container’s writable layer is **temporary**. If you remove the container, the data stored inside it disappears. That’s fine for stateless apps, but not for databases, uploads, logs, or anything users care about.

**Solution:** Store data outside the container lifecycle using **volumes**.

---

## Storage Options in Docker

### 1. Named Volumes (Managed by Docker)

* Docker stores them in `/var/lib/docker/volumes/...` (Linux).
* Portable across machines (via backup/restore), decoupled from host paths.
* Best choice for **databases** and **production data**.

### 2. Bind Mounts (Host Path → Container Path)

* You choose the host folder; Docker mounts it into the container.
* Great for **local development** (edit on host, container sees changes).

### 3. tmpfs Mounts (RAM-Only, Linux)

* Data lives in **memory only**; disappears on restart.
* Useful for **secrets** or **caches** that shouldn’t hit disk.

---

## Quick Comparison

| Feature              | Named Volume            | Bind Mount                | tmpfs           |
| -------------------- | ----------------------- | ------------------------- | --------------- |
| **Where data lives** | Docker-managed location | Your chosen host folder   | RAM only        |
| **Best for**         | Databases, durable data | Dev workflows, hot reload | Secrets, caches |
| **Portability**      | High                    | Host-path specific        | None            |
| **Performance**      | Consistent              | Depends on host FS        | Fast (memory)   |

---

## `-v` vs `--mount` (Use `--mount` for Clarity)

Both achieve similar results, but `--mount` is more **explicit and readable**.

### Short form (`-v` / `--volume`)

* Named volume: `-v myvol:/container/path[:options]`
* Bind mount: `-v /host/path:/container/path[:options]`

### Long form (`--mount`)

* Named volume: `--mount type=volume,source=myvol,target=/container/path,readonly`
* Bind mount: `--mount type=bind,source="$(pwd)/site",target=/usr/share/nginx/html,readonly`

**Tip:** Prefer `--mount` in scripts and docs. Use `-v` when typing quickly.

---

## DEMO 1 — Data Loss Without a Volume (MySQL)

Run MySQL without a volume, add a row, then remove the container to see data vanish.

### 1. Run MySQL without a volume

```bash
docker run -d \
  --name mysql-novolume \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=testdb \
  mysql:8
```

**Flags explained:**

* `-d` → run in background
* `--name` → gives a friendly name
* `-e` → sets environment variables

### 2. Create a table and insert data

```bash
docker exec -it mysql-novolume \
  mysql -uroot -proot -e "CREATE TABLE testdb.users (id INT, name VARCHAR(50));"

docker exec -it mysql-novolume \
  mysql -uroot -proot -e "INSERT INTO testdb.users VALUES (1, 'DevOps Team');"

docker exec -it mysql-novolume \
  mysql -uroot -proot -e "SELECT * FROM testdb.users;"
```

### 3. Remove the container (data disappears)

```bash
docker stop mysql-novolume
docker rm mysql-novolume
```

Re-run MySQL now — the table is gone because the data lived inside the removed container.

---

## DEMO 2 — Persistence With a Named Volume (MySQL)

Mount a **named volume** at MySQL’s data directory so data survives even after container removal.

### 1. Run MySQL with a named volume

```bash
docker run -d \
  --name mysql-volume \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=testdb \
  -v mysql_data:/var/lib/mysql \
  mysql:8
```

`-v mysql_data:/var/lib/mysql` creates (or reuses) a named volume `mysql_data` mounted at MySQL’s data path.

Confirm the volume exists:

```bash
docker volume ls

docker volume inspect mysql_data
```

### 2. Create and insert persistent data

```bash
docker exec -it mysql-volume \
  mysql -uroot -proot -e "CREATE TABLE testdb.users (id INT, name VARCHAR(50));"

docker exec -it mysql-volume \
  mysql -uroot -proot -e "INSERT INTO testdb.users VALUES (2, 'Developer Team');"
```

### 3. Remove and re-run container with same volume

```bash
docker stop mysql-volume
docker rm mysql-volume

docker run -d \
  --name mysql-volume2 \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=testdb \
  -v mysql_data:/var/lib/mysql \
  mysql:8

docker exec -it mysql-volume2 \
  mysql -uroot -proot -e "SELECT * FROM testdb.users;"
```

Data persists because it lives in `mysql_data`, outside the container lifecycle.

---

## DEMO 3 — Bind Mount for Live Editing (Nginx)

Bind mounts are perfect for **development**: edit files on your host and see changes instantly.

### 1. Make a local site folder

```bash
mkdir -p site
cat > site/index.html <<'EOF'
<!doctype html>
<html><body>
  <h1>Hello from a bind mount!</h1>
  <p>Edit this file on your host to see instant changes.</p>
</body></html>
EOF
```

### 2. Run Nginx with a bind mount (read-only)

```bash
docker run -d \
  --name nginx-demo \
  -p 8080:80 \
  -v "$(pwd)/site":/usr/share/nginx/html:ro \
  nginx:alpine
```

Open **[http://localhost:8080](http://localhost:8080)** — your page appears! Edit `site/index.html` and refresh.

**On Windows PowerShell:**

```bash
-v ${PWD}\site:/usr/share/nginx/html:ro
```

**Read-Only vs Read-Write:**

* Append `:ro` to make a mount read-only (good for configs).
* Default is `:rw` (read-write).

---

## Managing Volumes

```bash
# List volumes
docker volume ls

# Create a named volume
docker volume create mydata

# Inspect a volume
docker volume inspect mydata

# Remove a volume
docker volume rm mydata

# Remove all unused volumes
docker volume prune
```

> Note: Removing a container **does not remove** named volumes. Anonymous ones can be removed with `docker rm -v`.

---

## Backing Up & Restoring a Named Volume

Use a temporary container to `tar` the volume contents.

### Backup

```bash
docker run --rm \
  -v mysql_data:/data \
  -v "$(pwd)":/backup \
  alpine sh -c 'cd /data && tar -czf /backup/mysql_data_$(date +%F).tgz .'
```

### Restore

```bash
docker run --rm \
  -v mysql_data:/restore \
  -v "$(pwd)":/backup \
  alpine sh -c 'cd /restore && tar -xzf /backup/mysql_data_*.tgz'
```

This way, you can back up and restore data in volumes **without running full database exports**.

---

## Cheat Sheet

```bash
# Named volume (create + use)
docker volume create myvol
docker run -d --name c1 --mount type=volume,source=myvol,target=/data busybox

# Bind mount (host folder)
docker run -d --name web -p 8080:80 --mount type=bind,source="$(pwd)/site",target=/usr/share/nginx/html,readonly nginx:alpine

# List and inspect volumes
docker volume ls
docker volume inspect myvol

# Prune unused volumes
docker volume prune

# Backup a volume
docker run --rm -v myvol:/data -v "$(pwd)":/backup alpine sh -c 'cd /data && tar -czf /backup/backup.tgz .'

# Restore a volume
docker run --rm -v myvol:/data -v "$(pwd)":/backup alpine sh -c 'cd /data && tar -xzf /backup/backup.tgz'
```

---

## Summary

* **Containers are ephemeral; volumes persist data.**
* Use **named volumes** for databases/production, **bind mounts** for local dev, **tmpfs** for sensitive data.
* Prefer `--mount` for clarity.
* Manage with `docker volume` commands (`create`, `inspect`, `prune`, etc.).

---

## ✨ Next Up

In the next article, we’ll connect a two-tier Flask app (app + database) and combine **networking** with **volumes** for a realistic understanding of the concepts.

---

**Author:** *Siva Sai Papani*
**Article Source:** [Docker Volumes on Hashnode](https://docker-part4.hashnode.dev/docker-volumes-persistent-storage-the-easy-way)
