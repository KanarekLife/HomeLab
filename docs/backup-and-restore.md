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
