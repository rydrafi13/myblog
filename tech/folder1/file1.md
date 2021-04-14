# Ceph Installation & Clustering

<p></p><div style="text-align: center;"><a href="https://1.bp.blogspot.com/-UnufGKDfm5E/YHUXuXoECGI/AAAAAAAAArg/cZHMjZ7XBro4RIZijJ_gTWMX_LOiPVikACLcBGAsYHQ/s1365/Ceph.png"><img border="0" data-original-height="663" data-original-width="1365" height="280" src="https://1.bp.blogspot.com/-UnufGKDfm5E/YHUXuXoECGI/AAAAAAAAArg/cZHMjZ7XBro4RIZijJ_gTWMX_LOiPVikACLcBGAsYHQ/w640-h310/Ceph.png" width="640" /></a></div>

Nah kita lanjut ke Langkah Installasi dan Clustering Ceph, Kali ini kita akan menginstall Ceph pada Sistem Operasi Linux (Ubuntu 20.04).

Lab yang digunakan oleh kita kali ini adalah sebagai berikut.

<div class="separator" style="clear: both;"><a href="https://1.bp.blogspot.com/-TTtJlORuh48/YHUZTl4xTwI/AAAAAAAAAro/W7tYE6kg1wULrkPe20TVSDLIhN4f7J6rACLcBGAsYHQ/s0/Setup.png" style="display: block; padding: 1em 0px; text-align: center;"><img alt="" border="0" data-original-height="128" data-original-width="599" height="85" src="https://1.bp.blogspot.com/-TTtJlORuh48/YHUZTl4xTwI/AAAAAAAAAro/W7tYE6kg1wULrkPe20TVSDLIhN4f7J6rACLcBGAsYHQ/w400-h85/Setup.png" width="400" /></a></div>

<center><i>*untuk ip bisa disesuaikan sendiri asalkan dalam 1 network.</i></center>

## Langkah Sebelum Installasi

```note
Lakukan pada node ceph-admin, ceph-osd1, ceph-osd2

```

- Konfigurasi Hostname
```scss
    # vi /etc/hostname
```
- Konfigurasi Hosts
```scss
    # vi /etc/hosts
```
- Update & Upgrade
```scss
    # apt-get update && apt-get upgrade && apt-get dist-upgrade -y
```
- Konfigurasi Zona Waktu
```scss
    # timedatectl set-timezone Asia/Jakarta
```
- Konfigurasi SSH
```scss
    # vi /etc/ssh/sshd_config
    
    # change
    PermitRootLogin yes
```
jika sudah kemudian restart service sshd
```scss
    # systemctl restart sshd
```

## Konfigurasi SSH

```note
Lakukan pada node ceph-admin

```

- Generate Public Key Baru
```scss
    # ssh-keygen -q -N ""
```
- Konfigurasi config ssh
```scss
    # vi ~/.ssh/config

    # add
    Host ceph-admin
        Hostname ceph-admin
        User root
    Host ceph-osd1
        Hostname ceph-osd1
        User root
    Host ceph-osd2
        Hostname ceph-osd2
        User root
```
- Ubah permission
```scss
   # chmod 600 ~/.ssh/config 
```
- Copy Public Key ke Node Ceph lainnya 
```scss
    # ssh-copy-id ceph-admin
    # ssh-copy-id ceph-osd1
    # ssh-copy-id ceph-osd2
```

## Install Ceph

```note
Lakukan pada node ceph-admin

```

- Install Paket Ceph
```scss
    # for NODE in ceph-admin ceph-osd1 ceph-osd2 do 
      ssh $NODE "apt-get install ceph -y" 
      done
```
- Generate uuid Node admin
```scss
    # uuidgen
```
- Konfigurasi file ceph.conf
```scss
    # vi /etc/ceph/ceph.conf

    # add
    [global]
    # definisikan cluster network untuk monitoring
    cluster network = 172.30.1.0/24
    # definisikan public network
    public network = 172.30.1.0/24
    # definisikan UUID 
    fsid = 44319ff7-4bb2-4244-a39e-ea7cc33a2fd4
    # definisikan IP address untuk Monitor Daemon
    mon host = 172.30.1.10
    # definisikan Hostname untuk Monitor Daemon
    mon initial members = admin
    osd pool default crush rule = -1

    # mon.(nama Nodenya)
    [mon.admin]
    # definisikan Hostname untuk Monitor Daemon
    host = admin
    # definisikan IP address untuk Monitor Daemon
    mon addr = 172.30.1.10
    # allow to delete pools
    mon allow pool delete = true
```
- Membuat secret key untuk Cluster Monitor
```scss
    # ceph-authtool --create-keyring /etc/ceph/ceph.mon.keyring --gen-key -n mon. --cap mon 'allow *' 
```
- Membuat secret key untuk Cluster Admin
```scss
    # ceph-authtool --create-keyring /etc/ceph/ceph.client.admin.keyring --gen-key -n client.admin --cap mon 'allow *' --cap osd 'allow *' --cap mds 'allow *' --cap mgr 'allow *' 
```
- Membuat secret key untuk bootstrap
```scss
    # ceph-authtool --create-keyring /var/lib/ceph/bootstrap-osd/ceph.keyring --gen-key -n client.bootstrap-osd --cap mon 'profile bootstrap-osd' --cap mgr 'allow r'
```
- Mengimport secret key yang sudah dibuat tadi
```scss
    # ceph-authtool /etc/ceph/ceph.mon.keyring --import-keyring /var/lib/ceph/bootstrap-osd/ceph.keyring
    # ceph-authtool /etc/ceph/ceph.mon.keyring --import-keyring /etc/ceph/ceph.client.admin.keyring 
```
- Membuat Monitor Map
```scss
    # FSID=$(grep "^fsid" /etc/ceph/ceph.conf | awk {'print $NF'})
    # NODENAME=$(grep "^mon initial" /etc/ceph/ceph.conf | awk {'print $NF'}) 
    # NODEIP=$(grep "^mon host" /etc/ceph/ceph.conf | awk {'print $NF'}) 
    # monmaptool --create --add $NODENAME $NODEIP --fsid $FSID /etc/ceph/monmap
```
- Membuat Direktori untuk Monitor Daemon
```scss
    # mkdir /var/lib/ceph/mon/ceph-admin
```
- Mengasosiasikan key dan monmap ke Monitor Daemon
```scss
    # ceph-mon --cluster ceph --mkfs -i $NODENAME --monmap /etc/ceph/monmap --keyring /etc/ceph/ceph.mon.keyring 
    # chown ceph. /etc/ceph/ceph.* 
    # chown -R ceph. /var/lib/ceph/mon/ceph-admin /var/lib/ceph/bootstrap-osd 
    # systemctl enable --now ceph-mon@$NODENAME 
```
- Mengaktifkan Messenger v2 protocol
```scss
    # ceph mon enable-msgr2 
```
- Mengaktifkan Placement Group
```scss
    # ceph mgr module enable pg_autoscaler 
```
- Membuat Direktori untuk Manager Daemon
```scss
    # mkdir /var/lib/ceph/mgr/ceph-admin 
```
- Membuat Authentication key untuk manager Daemon
```scss
    # ceph auth get-or-create mgr.$NODENAME mon 'allow profile mgr' osd 'allow *' mds 'allow *' 
    # ceph auth get-or-create mgr.admin | tee /etc/ceph/ceph.mgr.admin.keyring 
    # cp /etc/ceph/ceph.mgr.admin.keyring /var/lib/ceph/mgr/ceph-admin/keyring  
```
- Mengubah permission file ceph.conf
```scss 
    # chown ceph. /etc/ceph/ceph.mgr.admin.keyring 
    # chown -R ceph. /var/lib/ceph/mgr/ceph-admin
```
- Restart service Ceph
```scss 
    # systemctl enable --now ceph-mgr@$NODENAME
```
- Verifikasi Cluster Status
```scss
    # ceph -s 
```
<div class="separator" style="clear: both; text-align: center;"><img border="0" data-original-height="218" data-original-width="425" height="164" src="https://1.bp.blogspot.com/-0FZiIx2Uv4Y/YHZbjtc8wDI/AAAAAAAAAr4/45FBbhHciMwCkBEBVMU16TVSN3wOruZGACLcBGAsYHQ/w320-h164/Ceph_mon.png" width="320" /></div>
<center><i>*disini kita sudah berhasil menjalankan ceph monitor, tetapi kita belum memiliki cluster storage.</i></center>

## Cluster Storage

```note
Lakukan pada node ceph-admin

```
Copy file konfigurasi dan keyring dari node **ceph-admin** ke node lainnya.
- Copy ke **ceph-osd1**
```scss
    # scp /etc/ceph/ceph.conf ceph-osd1:/etc/ceph/ceph.conf
    # scp /etc/ceph/ceph.client.admin.keyring ceph-osd1:/etc/ceph
    # scp /var/lib/ceph/bootstrap-osd/ceph.keyring ceph-osd1:/var/lib/ceph/bootstrap-osd  
```
- Copy ke **ceph-osd2**
```scss
    # scp /etc/ceph/ceph.conf ceph-osd2:/etc/ceph/ceph.conf
    # scp /etc/ceph/ceph.client.admin.keyring ceph-osd2:/etc/ceph
    # scp /var/lib/ceph/bootstrap-osd/ceph.keyring ceph-osd2:/var/lib/ceph/bootstrap-osd 
```
```note
Lakukan pada node ceph-admin, ceph-osd1, ceph-osd2

```
- Konfigurasi osd pada node **ceph-admin**
```scss
    # chown ceph. /etc/ceph/ceph.* /var/lib/ceph/bootstrap-osd/*; \
    parted --script /dev/sdb 'mklabel gpt'; \
    parted --script /dev/sdb "mkpart primary 0% 100%"; \
    ceph-volume lvm create --data /dev/sdb1
```
- Konfigurasi osd pada node **ceph-osd1**
```scss
    # chown ceph. /etc/ceph/ceph.* /var/lib/ceph/bootstrap-osd/*; \
    parted --script /dev/sdb 'mklabel gpt'; \
    parted --script /dev/sdb "mkpart primary 0% 100%"; \
    ceph-volume lvm create --data /dev/sdb1
```
- Konfigurasi osd pada node **ceph-osd2**
```scss
    # chown ceph. /etc/ceph/ceph.* /var/lib/ceph/bootstrap-osd/*; \
    parted --script /dev/sdb 'mklabel gpt'; \
    parted --script /dev/sdb "mkpart primary 0% 100%"; \
    ceph-volume lvm create --data /dev/sdb1
```

```note
Lakukan pada node ceph-admin

```

- Verifikasi Ceph Cluster
```scss
    # ceph -s
```