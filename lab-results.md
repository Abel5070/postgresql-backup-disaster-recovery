# PostgreSQL Backup & Disaster Recovery Lab — Results

## 1. Logical Backup

A PostgreSQL custom-format backup of the `bootcamp` database was created successfully.

The backup was verified using `pg_restore --list`, and it was restored into a separate database named `bootcamp_check`.

**Result:** Successful ✅

---

## 2. WAL Archiving and Base Backup

WAL archiving was enabled using:

```text
wal_level = replica
archive_mode = on
archive_command = 'cp %p /home/abel_nestor/backups/wal/%f'
```

A physical base backup was created using `pg_basebackup`.

The backup completed successfully:

```text
443299/443299 kB (100%), 1/1 tablespace
```

**Result:** Successful ✅

---

## 3. Point-in-Time Recovery

A simulated disaster was performed by deleting data from the `students` table.

Before the disaster, the recovery target time was recorded:

```text
2026-09-25 11:32:41.698602+03
```

The PostgreSQL base backup and archived WAL files were then used to perform point-in-time recovery.

After recovery, the `students` table was checked and the deleted record was successfully restored.

**Result:** Successful ✅

---

## 4. Streaming Standby

A replication role named `replicator` was created.

A standby PostgreSQL server was created using `pg_basebackup` and configured to run on port `5433`.

The standby was verified using:

```sql
SELECT pg_is_in_recovery();
```

The result was:

```text
t
```

This confirmed that the server was operating in recovery/standby mode.

**Result:** Successful ✅

---

## 5. Replication Health

Replication was checked from the primary PostgreSQL server using `pg_stat_replication`.

The standby showed:

```text
state = streaming
```

This confirmed that the primary server was successfully streaming WAL changes to the standby server.

**Result:** Successful ✅

---

# Final Assessment

All required parts of the PostgreSQL Backup and Disaster Recovery lab were completed successfully.

### Completed tasks

* ✅ Logical backup
* ✅ Backup verification
* ✅ Database restoration
* ✅ WAL archiving
* ✅ Physical base backup
* ✅ Point-in-time recovery
* ✅ Simulated disaster recovery
* ✅ Streaming replication
* ✅ Standby server verification
* ✅ Replication health monitoring

**Overall status: COMPLETED SUCCESSFULLY**
