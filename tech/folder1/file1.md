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
    
    ubah bagian root login
    PermitRootLogin yes
```
jika sudah kemudian restart service sshd
```scss
    # systemctl restart sshd
```

