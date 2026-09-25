# PostgreSQL Backup & Disaster Recovery Lab

## Overview

This lab demonstrates PostgreSQL backup and disaster recovery techniques, including logical backups, WAL archiving, point-in-time recovery (PITR), and streaming replication.

**PostgreSQL version:** 18.6
**Primary database:** `bootcamp`
**Primary port:** `5432`
**Standby port:** `5433`

---

## Step 1: Take and Verify a Logical Backup

Created the backup directory:

```bash
mkdir -p ~/backups
```

Created a PostgreSQL custom-format backup:

```bash
sudo -u postgres pg_dump -Fc -f /tmp/bootcamp.dump bootcamp
```

Copied the backup to the backups directory:

```bash
cp /tmp/bootcamp.dump ~/backups/bootcamp.dump
```

Verified the backup:

```bash
sudo -u postgres pg_restore --list /tmp/bootcamp.dump | head
```

Created a test database:

```bash
sudo -u postgres createdb bootcamp_check
```

Restored the backup:

```bash
sudo -u postgres pg_restore -d bootcamp_check /tmp/bootcamp.dump
```

The restored database contained the expected tables, including:

* `students`
* `orders`

---

## Step 2: Enable WAL Archiving

Configured PostgreSQL with:

```text
wal_level = replica
archive_mode = on
archive_command = 'cp %p /home/abel_nestor/backups/wal/%f'
```

Created the WAL archive directory:

```bash
mkdir -p ~/backups/wal
```

Set the correct ownership:

```bash
sudo chown -R postgres:postgres /home/abel_nestor/backups/wal
```

Because PostgreSQL runs as the `postgres` user, permission was granted to access the home directory:

```bash
sudo setfacl -m u:postgres:x /home/abel_nestor
```

PostgreSQL was restarted after changing the configuration.

A physical base backup was then created:

```bash
sudo -u postgres pg_basebackup -D /home/abel_nestor/backups/base -Ft -z -Xs -P
```

The backup completed successfully with:

```text
443299/443299 kB (100%), 1/1 tablespace
```

---

## Step 3: Point-in-Time Recovery

Before simulating the disaster, the recovery target time was recorded:

```text
2026-09-25 11:32:41.698602+03
```

The original `students` table contained one record.

The disaster was simulated by deleting the record:

```sql
DELETE FROM students;
```

The WAL was switched to make sure the changes were archived:

```bash
sudo -u postgres psql -c "SELECT pg_switch_wal();"
```

PostgreSQL was stopped and the original data directory was preserved.

The base backup was restored to the PostgreSQL data directory.

The recovery configuration was set to:

```text
restore_command = 'cp /home/abel_nestor/backups/wal/%f %p'
recovery_target_time = '2026-09-25 11:32:41.698602+03'
```

A recovery signal file was created:

```bash
sudo -u postgres touch /var/lib/postgresql/18/main/recovery.signal
```

PostgreSQL was then started.

The recovery log confirmed that PostgreSQL performed point-in-time recovery and stopped at the specified recovery target.

The `students` table was checked after recovery:

```sql
SELECT count(*) FROM students;
```

The original record was recovered successfully.

---

## Step 4: Set Up a Streaming Standby

Created a replication role:

```sql
CREATE ROLE replicator WITH REPLICATION LOGIN PASSWORD 'reppass';
```

Configured `pg_hba.conf` to allow replication from localhost:

```text
host    replication    replicator    127.0.0.1/32    scram-sha-256
```

Tested the replication user's authentication:

```bash
PGPASSWORD=reppass psql -h 127.0.0.1 -U replicator -d postgres -c "SELECT current_user;"
```

The result confirmed:

```text
current_user
-------------
replicator
```

Created the standby directory and set its permissions:

```bash
sudo chown postgres:postgres /home/abel_nestor/standby
sudo chmod 700 /home/abel_nestor/standby
```

Created the standby using:

```bash
sudo -u postgres env PGPASSWORD=reppass pg_basebackup \
-h 127.0.0.1 \
-U replicator \
-D /home/abel_nestor/standby \
-R \
-P
```

The backup completed successfully.

Configured the standby to use port `5433`:

```text
port = 5433
```

Copied the required `pg_hba.conf` file to the standby:

```bash
sudo cp /etc/postgresql/18/main/pg_hba.conf \
/home/abel_nestor/standby/pg_hba.conf
```

Started the standby:

```bash
sudo -u postgres /usr/lib/postgresql/18/bin/pg_ctl \
-D /home/abel_nestor/standby \
-l /home/abel_nestor/standby/standby.log \
-w start
```

Verified that the standby was in recovery mode:

```bash
sudo -u postgres psql -p 5433 \
-c "SELECT pg_is_in_recovery();"
```

The result was:

```text
t
```

This confirmed that the second PostgreSQL server was operating as a standby.

---

## Step 5: Check Replication Health

Replication health was checked from the primary server:

```bash
sudo -u postgres psql -p 5432 -c "
SELECT application_name, state,
       pg_wal_lsn_diff(sent_lsn, replay_lsn) AS lag_bytes
FROM pg_stat_replication;"
```

The standby appeared with:

```text
state = streaming
```

This confirmed that the primary PostgreSQL server was successfully streaming WAL changes to the standby server.

---

## Final Result

The PostgreSQL backup and disaster recovery lab was completed successfully.

The following were demonstrated:

* Logical database backup and restoration
* WAL archiving
* Physical base backup
* Point-in-time recovery
* Recovery from a simulated data loss event
* PostgreSQL streaming replication
* Standby server operation
* Replication health monitoring

**Final status: Completed successfully.**
