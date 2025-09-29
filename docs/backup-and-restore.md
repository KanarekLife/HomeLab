# Backup & Restore

## Creating Backup

```sh
$ velero backup create <backup> --snapshot-volumes --volume-snapshot-locations=default --storage-location=default
```

## Restoring Backup

```sh
# Remove namespaces, PVs, PVCs, ZFS volumes and everything that you want to restore from the backup
$ velero restore create --from-backup <backup> --restore-volumes=true
# Remove PVCs since they will be unbounded
$ velero restore create --from-backup <backup> --restore-volumes=false
```

## Instruction

Let's assume we want to restore authentik alongside its database.

1. Stop argocd autosync for both authentik and app-of-apps.
2. Delete authentik namespace (PVCs will be deleted as well), PVs and zfsvols that you want to restore from the backup.
3. Restore the backup with `velero restore create --from-backup <backup> --restore-volumes=true --include-namespaces authentik`. It should restore everything including PVCs, PVs and zfsvols.
