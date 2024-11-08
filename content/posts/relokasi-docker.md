+++
title = 'Relokasi Docker'
date = 2024-08-29T18:41:16+07:00
draft = false
categories = ['docker']
tags = ['dev']
+++

### Latar Belakang Masalah
Secara default, direktori docker terletak pada `/var/lib/docker`. Ada kalanya, ketika docker image , volume dan container sudah terlalu besar sehingga memakan storage. Kita ingin memindahkan letak direktori docker ini agar tidak memakan storage `root` kita yang pada akhirnya akan menghentikan service docker itu sendiri.

### Solusi
Maka dari itu, kita perlu merelokasi direktori default docker ini ke partisi lain yang memiliki kapasitas lebih besar daripada partisi `root`. 

### Persiapan

1. backup

`sudo mv /var/lib/docker /var/lib/docker.bak`

2. stop docker service

`sudo systemctl stop docker.service` \
`sudo systemctl stop docker.socker`

3. cek status docker service

`sudo systemctl status docker.service`

4. pindahkan direktori docker ke tempat baru

`sudo rsync -avxP /var/lib/docker/ /new/path/to/docker/`

Setelah berhasil ter-copy. Kita lanjutkan untuk merelokasi docker dengan cara membuat symbolic link.

### Membuat Symbolic Link
Sebenarnya, ada beberapa cara untuk merelokasi docker. Tapi, di bawah ini adalah cara yang paling mudah. Dikarenakan kita tidak perlu mengubah file konfigurasi docker.

1. membuat symbolic link

`sudo ln -s /new/path/to/docker/ /var/lib/docker`

2. start docker service

`sudo systemctl start docker`

3. cek status docker service untuk memastikan service berjalan tanpa masalah 

`sudo systemctl status docker`

4. hapus direktori docker yang lama, jika service docker sudah berhasil berjalan

`rm -r /var/lib/docker.bak`

### Penutup
Demikian cara merelokasi `/var/lib/docker` di sistem operasi linux. Cara ini sangat penting jika kita ingin menjaga partisi `root` kita tetap memiliki kapasitas storage yang cukup untuk aplikasi linux lainnya. 