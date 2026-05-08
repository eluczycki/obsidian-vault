coolify:supabase
When deploying supabase we have run into the following error:
 Container supabase-db-vmpkgro6mn7041tthd5xfk7l Started 
Error response from daemon: failed to create task for container: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: error during container init: exec: "/entrypoint.sh": permission denied

This is due to `entrypoint.sh` not being executable - quick fix:
ssh root@45.33.113.111 'chmod +x /data/coolify/services/vmpkgro6mn7041tthd5xfk7l/volumes/api/kong-entrypoint.sh'

