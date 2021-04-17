# Ceph : Integrasi Openstack Cinder Volume

Nah Selanjutnya kita akan mengintegrasikan Ceph storage dengan Openstack Block Storage (Cinder), nah Ceph Storage ini nanti bisa dijadikan sebagai Volume untuk Openstack.

## Prerequisite
```note
Lakukan pada node ceph-admin
```

- Konfigurasi Host
<br>nah kita tambahkan ip dan hostname dari node cinder dan compute
```
    # vi /etc/hosts
```

- Konfigurasi SSH
```
    # ssh-copy-id compute
    # ssh-copy-id cinder
```

## Install Ceph
```note
Lakukan pada node compute dan cinder
```

- Install ceph-common
```
    # apt-get install ceph-common
```

## Konfigurasi Ceph

- Copy file konfigurasi ceph
```note
Lakukan pada node ceph-admin
```
```
    # scp /etc/ceph/ceph.conf compute:/etc/ceph/
    # scp /etc/ceph/ceph.client.admin.keyring compute:/etc/ceph/
    # ssh compute "chown ceph. /etc/ceph/ceph."

    # scp /etc/ceph/ceph.conf cinder:/etc/ceph/
    # scp /etc/ceph/ceph.client.admin.keyring cinder:/etc/ceph/
    # ssh cinder "chown ceph. /etc/ceph/ceph."
```

- Setup Ceph Client Authentication
```note
Lakukan pada node ceph-admin
```
```
    # ceph auth get-or-create client.cinder mon 'profile rbd' osd 'profile rbd pool=volumes, profile rbd pool=vms, profile rbd-read-only pool=images' mgr 'profile rbd pool=volumes, profile rbd pool=vms'
    # ceph auth get-or-create client.cinder | ssh cinder sudo tee /etc/ceph/ceph.client.cinder.keyring
    # ssh cinder sudo chown cinder:cinder /etc/ceph/ceph.client.cinder.keyring
    # ceph auth get-or-create client.cinder | ssh compute sudo tee /etc/ceph/ceph.client.cinder.keyring
    # ceph auth get-key client.cinder | ssh compute tee client.cinder.key
```

- Konfigurasi file ceph
```note
Lakukan pada node cinder
```

```
    # vi /etc/cinder/cinder.conf
   
    # add
    [DEFAULT]
    ...
    enabled_backends = ceph
    glance_api_version = 2
    ...
    [ceph]
    volume_driver = cinder.volume.drivers.rbd.RBDDriver
    volume_backend_name = ceph
    rbd_pool = volumes
    rbd_ceph_conf = /etc/ceph/ceph.conf
    rbd_flatten_volume_from_snapshot = false
    rbd_max_clone_depth = 5
    rbd_store_chunk_size = 4
    rados_connect_timeout = -1
```

```note
Lakukan pada node compute 
```

- Generate UUID pada node compute
<br>kemudian copy uuid yang sudah digenerate
```
    # uuidgen 
```

- Membuat script untuk key libvirt
- Mendefinisikan script ke dalam libvirt
```
    # cat > secret.xml <<EOF
    <secret ephemeral='no' private='no'>
    <uuid>457eb676-33da-42ec-9a8c-9293d545c337</uuid>
    <usage type='ceph'>
        <name>client.cinder secret</name>
    </usage>
    </secret>
    EOF
    # sudo virsh secret-define --file secret.xml
    # sudo virsh secret-set-value --secret 457eb676-33da-42ec-9a8c-9293d545c337 --base64 $(cat client.cinder.key) && rm client.cinder.key secret.xml
```

## Verifikasi
```note
Lakukan pada node controller
```

- Membuat Volume
- Verfikasi Volume

<i>*Selesai</i>

```
Jika kamu tidak tahan dengan lelahnya belajar maka kamu harus tahan dengan perihnya kebodohan.
```