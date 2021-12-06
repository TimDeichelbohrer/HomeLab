# HomeLab

HomeLab Server & Desktop Configuration

- Server: Debian Stable /w K3s & ZFS
- Desktop: Pop!_OS Latest

  - Manual Patches Applied

    - <https://github.com/ArthurVardevanyan/pop-shell>
    - <https://github.com/ArthurVardevanyan/pop-cosmic>

## Extra Commands & Notes

### Desktop

```bash
git merge --no-ff
scp -r /home/arthur/vmware windowsBackup@10.0.0.3:/backup/WindowsBackup/vmware
7z a -t7z -m0=lzma2 -mx=9 -mfb=128 -md=256m -ms=on FOLDER.7z FOLDER
sudo sensors-detect
```

#### Gnome

Manually Install Extensions from extensions.gnome.org

- gnome-shell-extension-netspeed
- gnome-shell-extension-places-menu
- gnome-shell-extension-transparentnotification
- gnome-shell-extension-tray-icons-reloaded

#### Cura

Config files need to be applied manually.

```bash
machineConfigs/home/arthur/cura
```

#### GWE

Database file needs to be restored manually.

### Server

#### Ansible

```bash
ansible-playbook -i ansible/inventory --ask-become-pass ansible/server.yaml --ask-pass --check
ansible-playbook -i ansible/inventory --ask-become-pass ansible/desktop.yaml --ask-pass --check
```

#### ZFS

```bash
sudo zfs get compressratio
/usr/sbin/zfs send -i backup/File_Storage backup/File_Storage@20211201 | pv | ssh arthur@10.0.0.4 /usr/sbin/zfs receive -F backup/File_Storage
zfs send backup/File_Storage@20211101 | ssh arthur@10.0.0.4 zfs receive -F backup/File_Storage
```

Creating ZFS zvol for Timeshift

```bash
sudo zfs create -V 76.76G backup/Timeshift
sudo mkfs.ext4 /dev/zd0 # Note UUID
sudo mount /dev/zd0 /media/arthur/Timeshift
sudo umount /media/arthur/Timeshift
```

#### Kubernetes

<https://k3s.io/> <https://upcloud.com/community/tutorials/deploy-kubernetes-dashboard/>

```bash
watch $(echo "kubectl get pods -A -o wide |  grep -v 'svclb' | sort -k8 -r")
kubectl drain k3s-worker --ignore-daemonsets --delete-emptydir-data

curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server --disable traefik --flannel-iface=enp1s0 --kubelet-arg system-reserved=cpu=250m,memory=250M --kubelet-arg kube-reserved=cpu=250m,memory=200M" INSTALL_K3S_CHANNEL=v1.22 sh
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC=--flannel-iface=enp6s0 K3S_URL=https://10.0.0.3:6443 K3S_TOKEN=$K3S_TOKEN INSTALL_K3S_CHANNEL=v1.22 sh -
```

#### Cockpit

```bash
sudo nano /etc/cockpit/ws-certs.d/1-my-cert.cert
```

#### GitLab

```bash
gitlab-ctl registry-garbage-collect
gitlab-ctl reconfigure
```

#### Database

```sql
CREATE USER 'arthur'@'10.0.0.X' IDENTIFIED BY 'arthur';
GRANT ALL PRIVILEGES ON *.* TO `arthur`@`10.0.0.X`;

FLUSH PRIVILEGES;

# % for everything
CREATE USER 'spotifyTest'@'10.42.0.%' IDENTIFIED BY 'spotifyTest';
GRANT ALL PRIVILEGES ON spotifyTest.* TO `spotifyTest`@`10.42.0.%`;

# View Only Access
GRANT SELECT, LOCK TABLES, SHOW VIEW ON *.* TO 'backup'@'10.42.0.1' IDENTIFIED BY 'backup';
```
