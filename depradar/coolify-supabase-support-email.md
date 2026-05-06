TO: support@coolify.io
SUBJECT: Bug Report: Supabase service deployment creates entrypoint/config files as directories causing container start failures

---

Hi Coolify team,

Ran into an issue deploying Supabase as a service via the Coolify UI. Sharing the details in case it's useful for your team or if there's a known workaround.

## The Problem

When deploying the "Supabase" service type (not "Postgres Supabase"), several container entrypoint scripts fail to start with permission denied errors:

```
OCI runtime create failed: runc create failed: unable to start container process: error during container init: exec: "/home/kong/kong-entrypoint.sh": permission denied
```

And for another container:

```
error mounting ".../pooler.exs" to rootfs at "/etc/pooler/pooler.exs": mount src=..., Are you trying to mount a directory onto a file (or vice-versa)?
```

## Root Cause (Observation)

The issue appears to be that Coolify creates **empty directories** at volume mount host paths instead of files. When Docker tries to bind-mount these paths, it fails because:
1. The entrypoint scripts don't have execute permissions (created as 644)
2. Some config files are created as directories instead of files

## Affected Containers
- `supabase-kong` — `kong-entrypoint.sh` missing execute bit
- `minio-createbucket` — `entrypoint.sh` created as empty directory
- `supabase-supavisor` — `pooler.exs` created as directory instead of file

## Environment
- Server: Ubuntu on Linode (4GB → upgraded to 8GB)
- Coolify: Latest cloud version
- Supabase: Latest official templates
- Memory was NOT the issue — server was at ~18% RAM during deploy

## Workaround Applied
Manually fixed permissions and replaced directories with proper files, then ran `docker compose up -d` to restart affected containers.

## Questions
- Is this a known issue with the Supabase service template?
- Is there something in our deployment config that could prevent this?
- Should I file this on GitHub Issues?

Happy to provide more details if helpful.

Thanks,
freshmaker