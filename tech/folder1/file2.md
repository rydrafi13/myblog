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
## Prerequisite
```note
Lakukan pada node ceph-admin
```

- Konfigurasi Host
<br>nah kita tambahkan ip dan hostname dari node controller
```
    # vi /etc/hosts
```

- Konfigurasi SSH
```
    # ssh-copy-id controller
```

## Installasi Ceph
```note
Lakukan pada node controller
```
- Install ceph-common
```
    # apt-get install ceph-common
```

## Konfigurasi Ceph
- Copy file konfigurasi file

```note
Lakukan pada node ceph-admin
```

```
    # scp /etc/ceph/ceph.conf controller:/etc/ceph/
    # scp /etc/ceph/ceph.client.admin.keyring controller:/etc/ceph/
    # ssh controller "chown ceph. /etc/ceph/ceph."
```

- Setup Ceph Client Authentication 

```note
Lakukan pada node ceph-admin
```
```
    # ceph auth get-or-create client.glance mon 'profile rbd' osd 'profile rbd pool=images' mgr 'profile rbd pool=images'
    # ceph auth get-or-create client.glance | ssh controller sudo tee /etc/ceph/ceph.client.glance.keyring
    # ssh controller sudo chown glance:glance /etc/ceph/ceph.client.glance.keyring
```

```note
Lakukan pada node controller
```
- Konfigurasi file ceph
```
    # vi /etc/ceph/ceph.conf

    # add
    [client.images] 
    keyring = /etc/ceph/ceph.client.glance.keyring
```
- Konfigurasi file glance
```
    # vi /etc/glance/glance-api.conf

    # add
    [default]
    show_image_direct_url = True

    [glance_store]
    stores = rbd
    default_store = rbd
    rbd_store_pool = images
    rbd_store_user = glance
    rbd_store_ceph_conf = /etc/ceph/ceph.conf
    rbd_store_chunk_size = 8
```
- Restart service glance
```
    # systemctl restart glance-api
```

## Verifikasi
```note
Lakukan pada node controller
```

- Upload Image Glance
```
    # openstack image create --file cirros-0.5.1-x86_64-disk.img --container-format bare --disk-format raw --public Ceph-Cirros
```
- Verifikasi Image
```note
Lakukan pada node controller
```

```
    # openstack image list
```
<div class="separator" style="clear: both; text-align: center;"><a href="https://1.bp.blogspot.com/-WPi6ItT6F_w/YHpW8gilO5I/AAAAAAAAAsg/Cy_ZbL_U6o4YgjeFOFfcyE2tIM6gguNUwCLcBGAsYHQ/s446/Screenshot%2Bfrom%2B2021-04-17%2B10-27-57.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="102" data-original-width="446" height="73" src="https://1.bp.blogspot.com/-WPi6ItT6F_w/YHpW8gilO5I/AAAAAAAAAsg/Cy_ZbL_U6o4YgjeFOFfcyE2tIM6gguNUwCLcBGAsYHQ/w320-h73/Screenshot%2Bfrom%2B2021-04-17%2B10-27-57.png" width="320" /></a></div><br />

```
    # rbd -p images ls 
```
<div class="separator" style="clear: both; text-align: center;"><a href="https://1.bp.blogspot.com/-3PeU5zDy_W8/YHpXSQgGvZI/AAAAAAAAAso/qpPt39MNUWEllr07G9_AmFLu1UKfnK1xwCLcBGAsYHQ/s353/Screenshot%2Bfrom%2B2021-04-17%2B10-34-02.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="35" data-original-width="353" src="https://1.bp.blogspot.com/-3PeU5zDy_W8/YHpXSQgGvZI/AAAAAAAAAso/qpPt39MNUWEllr07G9_AmFLu1UKfnK1xwCLcBGAsYHQ/s320/Screenshot%2Bfrom%2B2021-04-17%2B10-34-02.png" width="320" /></a></div><br />

```
    # rbd -p images info <id_image>
```
<div class="separator" style="clear: both; text-align: center;"><a href="https://1.bp.blogspot.com/-yZ2iVnZPxbQ/YHpXiAyDAAI/AAAAAAAAAsw/lRqOyA6XXpsqgRPzXbkTblSQVu5ED0hfgCLcBGAsYHQ/s626/Screenshot%2Bfrom%2B2021-04-17%2B10-35-09.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="212" data-original-width="626" src="https://1.bp.blogspot.com/-yZ2iVnZPxbQ/YHpXiAyDAAI/AAAAAAAAAsw/lRqOyA6XXpsqgRPzXbkTblSQVu5ED0hfgCLcBGAsYHQ/s320/Screenshot%2Bfrom%2B2021-04-17%2B10-35-09.png" width="320" /></a></div><br />

<i>*Selesai</i>

```
Jika kamu tidak tahan dengan lelahnya belajar maka kamu harus tahan dengan perihnya kebodohan.
```