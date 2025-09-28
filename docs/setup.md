# HomeLab Setup

- Go to [Image Factory Talos Linux](https://factory.talos.dev/) and recreate the following configuration:
    - Machine Type: 'Bare Metal'
    - Version: '1.11.0'
    - Architecture: 'amd64' (Secure Boot: Enabled)
    - Extensions: `siderolabs/nvidia-container-toolkit-production (570.172.08-v1.17.8)`, `siderolabs/nonfree-kmod-nvidia-production (570.172.08-v1.11.0)`, `siderolabs/intel-ucode (20250812)`, `siderolabs/zfs (2.3.3-v1.11.0)`
    - ID Schema: `fb9b2a57f871f665c4e9ebc57abeefd4139e6a3c96c3f51a8688b34a6f173239`

- Download the ISO and boot the server from it.

- Setup `talosctl` on your local machine by following the [official guide](https://www.talos.dev/v1.11/introduction/getting-started/).

- Create `homelab.lan.nieradko.com` DNS A record pointing to your server IP.

- Execute the following commands on your local machine (adjusting parameters to your needs):

```bash
$ CONTROL_PLANE_IP=homelab.lan.nieradko.com
$ talosctl get disks -n homelab.lan.nieradko.com --insecure
$ mkdir -p ./talos && cd ./talos
$ talosctl gen config homelab.nieradko.com https://homelab.lan.nieradko.com:6443 --install-disk /dev/disk/by-id/nvme-RPFTJ256PDD2MWX_SS0R27339Z1CD04L130X --install-image factory.talos.dev/metal-installer-secureboot/fb9b2a57f871f665c4e9ebc57abeefd4139e6a3c96c3f51a8688b34a6f173239:v1.11.0 --additional-sans homelab.lan.nieradko.com

# Edit controlplane.yaml to your needs (e.g. networking, ntp, users, etc.)
# 1. Enable     allowSchedulingOnControlPlanes: true
# 2. Enable kernel modules ( nvidia, nvidia_uvm, nvidia_drm, nvidia_modeset, zfs )

$ talosctl apply-config --insecure --nodes homelab.lan.nieradko.com --file controlplane.yaml
$ mv talosconfig ~/.talos/config
# Edit talosconfig to include endpoint and node
$ talosctl --talosconfig=./talosconfig config endpoints homelab.lan.nieradko.com
$ talosctl bootstrap --nodes homelab.lan.nieradko.com --talosconfig talosconfig
$ talosctl kubeconfig --nodes homelab.lan.nieradko.com --talosconfig talosconfig
$ talosctl health --nodes homelab.lan.nieradko.com --talosconfig talosconfig
$ talosctl config merge talosconfig
$ cd ../ && rm -rf ./talos
$ helm repo add argo https://argoproj.github.io/argo-helm
$ helm repo update
$ helm install argocd argo/argo-cd -f kubernetes/argocd/values.yaml --create-namespace --namespace argocd --atomic
$ kubectl apply -f kubernetes/argocd/apps/root-app.yaml
```

## ZFS Setup

```bash
$ kubectl apply -f kubernetes/openebs/shell-pod.yaml
$ kubectl exec -n kube-system pod/zfs-shell -- nsenter --mount=/proc/1/ns/mnt -- zpool create -m legacy -f zfspv-pool mirror /dev/disk/by-id/ata-ST4000VN008-2DR166_ZGY94AQV /dev/disk/by-id/ata-ST4000VN008-2DR166_ZGY94AJ3
$ kubectl exec -n kube-system pod/zfs-shell -- nsenter --mount=/proc/1/ns/mnt -- zpool status
$ kubectl exec -n kube-system pod/zfs-shell -- nsenter --mount=/proc/1/ns/mnt -- zfs list
```
