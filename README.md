# Ansible Playbooks

Playbooks untuk maintenance, monitoring, deployment, dan sertifikat. Repository ini dipakai dari Semaphore; inventory dan secret tetap dikelola di Semaphore/environment, bukan disimpan di repo.

## Struktur

```text
playbooks/
  deploy/       # deployment aplikasi
  maintenance/  # package, certificate, service updates
  monitoring/   # health dan capacity checks
shared/
  tasks/        # task fragments reusable
  vars/         # connection defaults reusable
Archive/        # playbooks retired; tidak dipakai
```

## Entry playbooks

| Fungsi | Path |
| --- | --- |
| Disk-space monitoring | `playbooks/monitoring/check-disk-space.yml` |
| APT upgrade and cleanup | `playbooks/maintenance/upgrade-apt-packages.yml` |
| Cloudflared update | `playbooks/maintenance/update-cloudflared.yml` |
| Dozzle update | `playbooks/maintenance/update-dozzle.yml` |
| Let's Encrypt renewal | `playbooks/maintenance/renew-certificates.yml` |
| Proxmox storage health | `playbooks/monitoring/check-proxmox-storage-health.yml` |
| Hexatech deployment | `playbooks/deploy/deploy-hexatech.yml` |

`shared/tasks/` bukan entry point Semaphore. File tersebut dipanggil oleh playbook yang membutuhkan task connectivity, prerequisite host, atau lifecycle Proxmox VM/LXC.

## APT upgrade di LXC

`upgrade-apt-packages.yml` mendeteksi host LXC lewat `ansible_virtualization_type`. Paket Docker yang terpasang (`docker.io`, Docker CE/plugin, `containerd`, dan `runc`) di-hold sementara selama upgrade, lalu dikembalikan ke state hold sebelumnya. Host non-LXC tetap mendapat upgrade normal.

## Disk-space monitoring

`check-disk-space.yml` aman untuk emergency check pada Proxmox host, guest VM/LXC, dan VPS. Ia memakai `raw df` tanpa facts, APT, Python, atau write ke host. Jika `df` gagal, Semaphore mengirim alert kegagalan ke `ADW_NOTIF` dari controller.

`check-proxmox-storage-health.yml` khusus Proxmox host. Ia alert `ADW_NOTIF` saat ZFS pool tidak `ONLINE`, mdraid degraded, atau SMART melaporkan status gagal. Tidak alert bila ZFS, mdraid, atau `smartctl` tidak tersedia.

## Dependencies

`update-dozzle.yml` memerlukan collection `community.docker`:

```bash
ansible-galaxy collection install -r requirements.yml
```

## Semaphore migration

Update template Semaphore ke path pada tabel di atas sebelum run berikutnya. Playbook lama di root sudah tidak tersedia. `Archive/` sengaja tidak disentuh dan tidak termasuk workflow aktif.
