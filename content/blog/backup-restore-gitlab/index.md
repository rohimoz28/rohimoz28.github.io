+++
title = 'Backup dan Restore Gitlab Dengan Docker Pada Server Berbeda'
date = '2024-12-24T11:52:59+07:00'
draft = false
description = 'Implementasi Backup dan Restore GitLab menggunakan Docker Pada Server Berbeda'
categories= ['Dev Ops']
tags = ['git', 'gitlab']
+++

## Latar Belakang Masalah
Adakalanya, karena beberapa masalah, jadi mengharuskan kita untuk memindahkan GitLab ke server lain. Jika ingin melihat garis besar cara 
backup dan restore GitLab, teman-teman bisa lihat dokumentasinya langsung di [Overview Backup Restore Gitlab](https://docs.gitlab.com/ee/administration/backup_restore/).

Atau, jika ingin langsung praktik, bisa mengikuti Tutorial berikut pada artikel ini.

## Prerequisites
* GitLab harus memiliki versi yang sama.
* Asumsi bahwa GitLab sudah terinstall di kedua server menggunakan docker.
* Storage yang mencukupi, karena file backup GitLab cukup besar (2GB ++).

## Backup
```
# Lokasi default file backup di /var/opt/gitlab/backups
docker exec -t <name of container> gitlab-rake gitlab:backup:create

# copy file hasil backup, dari container ke host machine
docker cp <name of container>:/var/opt/gitlab/backups/<name of backup> .

# backup file gitlab-secrets.json
docker cp <name of container>:/etc/gitlab/gitlab-secrets.json .

# backup file gitlab.rb
docker cp <name of container>:/etc/gitlab/gitlab.rb .
```

Kemudian, file hasil backup dijadikan satu kedalam sebuah folder, misalkan `backup_dir`. 
Lalu lakukan compress dengan command : ```tar -zcvf backup_dir.tar.gz backup_dir```

## Migrasi
Pertama, kita harus login sebagai user di server yang nantinya akan menjadi tempat restore GitLab.

```
scp yourUsername@yourIPAddress:/path/to/compressed/backup/file /path/to/backup/folder
```

## Restore
```
# copy file backup ke lokasi default di dalam container
docker cp <name of backup> <name of container>:/var/opt/gitlab/backups

# ubah user permission untuk file backup
docker exec -it <name of container> chown git:git /var/opt/gitlab/backups/<name of backup file>

# jalankan command untuk restore file backup
docker exec -it <name of container> gitlab-rake gitlab:backup:restore

# restore gitlab-secrets.json
docker cp gitlab-secrets.json <name of container>:/etc/gitlab/gitlab-secrets.json

# restore gitlab.rb
docker cp gitlab.rb <name of container>:/etc/gitlab/gitlab.rb

# restart gitlab command
docker exec -it <name of container> gitlab-ctl reconfigure
```

## Contoh Kasus Ketika Restore Gitlab
### Masalah File Permission
Secara default, Gitlab dan folder-folder didalamnya punya konfigurasi default yang mungkin saja berubah ketika melakukan proses backup dan
restore. Untuk itu, kita bisa mengembalikan settingan default dari konfigurasi folder-folder Gitlab tersebut dengan command berikut.
```
docker exec -it <name of container> update-permissions
docker restart <name of container>
```

### Restore Berhasil Tapi Belum Ada Perubahan Pada Gitlab
Setelah restore berhasil dan kita restart docker gitlab-nya, seharusnya sudah tampak project-project dari hasil backup kita. Namun, jika project-project
tersebut belum muncul setelah proses restore berhasil. Bisa coba command di bawah ini untuk restart Gitlab.
```
docker exec -it <name of container> gitlab-ctl stop
docker exec -it <name of container> gitlab-ctl start
```

## Backup Otomatis (Opsional)
Backup otomatis disini akan menggunakan sebuah shell script yang nantinya akan dijalankan otomatis menggunakan `cronjob`.
Berikut isi dari file script tersebut.
```
#/bin/sh
#script to automate the gitlab backup and save it in folder /home/gitlab-backup
docker exec -it gitlab gitlab-rake gitlab:backup:create
docker cp gitlab:/var/opt/gitlab/backups/ /path/to/gitlab-backup
docker cp gitlab:/etc/gitlab/gitlab.rb /path/to/gitlab-backup
docker cp gitlab:/etc/gitlab/gitlab-secrets.json /path/to/gitlab-backup
```

## Kesimpulan
Pada artikel ini, dijelaskan tata cara untuk melakukan backup, transfer file backup ke server berbeda, restore file backup menggunakan docker, serta
cara membuat script shell untuk keperluan backup otomatis.

Terimakasih.