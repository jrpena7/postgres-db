# Database Stack — Q&A

A running log of questions and decisions made while building the PostgreSQL + pgAdmin + Backup stack with Docker Compose.

---

## Table of Contents

1. [Will `init-db.sh` re-run on every container restart and cause conflicts?](#1-will-init-dbsh-re-run-on-every-container-restart-and-cause-conflicts)

---

### 1. Will `init-db.sh` re-run on every container restart and cause conflicts?

**Context:** The postgres service mounts an initialization script via:

```yaml
volumes:
  - ./scripts/init-db.sh:/docker-entrypoint-initdb.d/init-db.sh:ro
```

The concern was: if the container restarts and `init-db.sh` tries to create users and databases that already exist (because data is persisted on the host at `./data/postgres`), would it fail or cause inconsistencies?

**Answer:** No. The official `postgres` Docker image only executes scripts inside `/docker-entrypoint-initdb.d/` when the data directory is **completely empty** — that is, on the very first initialization. On every subsequent restart, if `./data/postgres` already contains data, all init scripts are **silently skipped**.

The lifecycle is:

```
First run
  └─► ./data/postgres is empty
        └─► PostgreSQL initializes the data directory
              └─► runs init-db.sh  →  creates users, databases, etc.

Every subsequent restart
  └─► ./data/postgres already has data
        └─► PostgreSQL just starts
              └─► init-db.sh is NOT executed
```

**Conclusion:** Mounting a persistent host volume and using init scripts are fully compatible. There is no risk of duplicate user/database creation errors on restart.