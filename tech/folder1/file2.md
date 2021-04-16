# Ceph : Integrasi Openstack Glance

Nah setelah kita berhasil melakukan installasi dan clustering Ceph, selanjutnya kita akan melakukan integrasi Ceph Storage dengan Openstack Image Service (Glance), nah Ceph Storage ini nanti sebagai tempat penyimpanan untuk glance.

## Membuat Pool 
```note
Lakukan pada node ceph-admin
```

- Membuat pool

```
    # ceph osd pool create volumes
    # ceph osd pool create images
    # ceph osd pool create backups
    # ceph osd pool create vms 
```
- Melakukan pool init

```
    # rbd pool init volumes
    # rbd pool init images
    # rbd pool init backups
    # rbd pool init vms
```
# Prerequisite
```note
Lakukan pada node ceph-admin
```

- Konfigurasi Host
- Konfigurasi SSH

# Installasi Ceph
```note
Lakukan pada node controller
```

# Konfigurasi Ceph
```note
Lakukan pada node ceph-admin
```

- Setup Ceph Client Authentication 

```note
Lakukan pada node controller
```

- Konfigurasi file ceph
- Konfigurasi file glance

- Restart service glance

# Verifikasi
```note
Lakukan pada node controller
```

- Upload Image Glance
- Verifikasi Image

<i>*Selesai</i>

```
Jika kamu tidak tahan dengan lelahnya belajar maka kamu harus tahan dengan perihnya kebodohan.
```