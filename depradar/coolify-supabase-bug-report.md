# Coolify Supabase Deployment Bug Report

## Summary
When deploying Supabase as a service via Coolify UI, container entrypoint scripts fail to start with `permission denied` errors. This affects multiple containers across both staging and production environments.

---

## Bug Description

### Error Messages

**1. Entrypoint script missing execute permission:**
```
OCI runtime create failed: runc create failed: unable to start container process: error during container init: exec: "/home/kong/kong-entrypoint.sh": permission denied
```

**2. Volume mount destination is a directory instead of a file:**
```
error mounting "/data/coolify/services/xxx/volumes/pooler/pooler.exs" to rootfs at "/etc/pooler/pooler.exs": mount src=..., dst=..., flags=MS_BIND|MS_REC: not a directory: Are you trying to mount a directory onto a file (or vice-versa)?
```

### Affected Containers
- `supabase-kong` — `kong-entrypoint.sh` missing execute bit
- `minio-createbucket` — entrypoint.sh created as empty directory
- `supabase-supavisor` — `pooler.exs` created as directory instead of file

---

## Root Cause
Coolify's service deployment process creates volume-mounted entrypoint/config files as **empty directories** instead of files with execute permissions. When Docker tries to mount these "files," it finds a directory at the host path and fails.

This appears to be a Coolify bug in how it initializes volume mounts for Supabase service containers.

---

## Environment
- **Server:** Linode 45.33.113.111
- **Coolify Version:** 4.0.0 (inferred from labels)
- **Supabase Version:** Various images (postgres:15.8.1.085, kong:3.9.1, gotrue:v2.186.0, etc.)
- **RAM:** 8GB (upgraded from 4GB — memory was NOT the issue)
- **Server Memory at deploy:** ~18% used — well within capacity

---

## Manual Fix Applied

### 1. Fix entrypoint.sh files (missing execute bit):
```bash
chmod +x /data/coolify/services/<uuid>/volumes/api/kong-entrypoint.sh
```

### 2. Fix minio-createbucket entrypoint (created as directory):
```bash
rm -rf /data/coolify/services/<uuid>/entrypoint.sh
echo "# minio/mc entrypoint" > /data/coolify/services/<uuid>/entrypoint.sh
chmod +x /data/coolify/services/<uuid>/entrypoint.sh
```

### 3. Fix supabase-supavisor pooler.exs (created as directory):
```bash
rm -rf /data/coolify/services/<uuid>/volumes/pooler/pooler.exs
echo "# Supavisor pooler config" > /data/coolify/services/<uuid>/volumes/pooler/pooler.exs
chmod +x /data/coolify/services/<uuid>/volumes/pooler/pooler.exs
```

### 4. Restart containers via docker compose:
```bash
cd /data/coolify/services/<uuid>
docker compose up -d
```

**Note:** `docker compose up -d` (with hyphen) recreates containers with proper configs. Using `docker start` alone after fixing files may leave containers in inconsistent state.

---

## Reproduction Steps

1. Go to Coolify UI → Project → Add Resource
2. Search for and select "Supabase" (NOT "Postgres Supabase")
3. Deploy with default settings
4. Observe containers in "Created" state with entrypoint permission errors

---

## Expected Behavior
All Supabase service containers should start successfully without manual intervention.

---

## Additional Notes
- The issue reproduced consistently across both staging and production deployments
- Memory was upgraded from 4GB to 8GB preemptively — the issue existed at both memory levels
- Kong container network also needed fixing on first staging deploy (was on `bridge` instead of service network), but this resolved itself on subsequent `docker compose up` calls

---

## Date Reported
2026-05-06