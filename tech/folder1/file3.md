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

- Install Ceph
```note
Lakukan pada node compute dan cinder
```

- Install ceph-common
```
    # apt-get install ceph-common
```

## Konfigurasi Ceph



<i>*Selesai</i>

```
Jika kamu tidak tahan dengan lelahnya belajar maka kamu harus tahan dengan perihnya kebodohan.
```