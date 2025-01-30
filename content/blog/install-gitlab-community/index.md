+++
title = 'Install Gitlab Community Menggunakan Docker'
date = '2025-01-30T11:49:28+07:00'
draft = false
description = 'Dengan beberapa alasan, adakalanya kita ingin memiliki Gitlab yang terinstall pada server yang kita kelola sendiri.'
categories= ['Dev Ops']
tags = ['gitlab','git']
+++
## Latar Belakang Masalah
Sebelumnya, saya menggunakan Gitlab.com (Enterprise). Namun belum menggunakan versi berbayarnya. Jadi, ada banyak limitasi di sana-sini. Seperti limitasi jumlah member
per project. Untuk itu, saya mencari solusi, dikarenakan perusahaan memiliki beberapa server, Gitlab Community Edition menjadi pilihan. 

## Requirement
- 2.5 GB storage
- 8 vCPU
- 16 GB RAM

Untuk lebih lengkapnya, silahkan kunjungi [official page](https://docs.gitlab.com/ee/install/requirements.html).

## Environment
Untuk menginstall Gitlab Community Edition. Saya akan menggunakan docker. Untuk image docker tersebut, tersedia [disini](https://hub.docker.com/r/gitlab/gitlab-ce).

## Docker Compose
Selanjutnya, kita akan membuat file konfigurasi `docker-compose.yaml`. Yang perlu di garis bawahi, kita akan membuat volume
docker dengan cara `bind` ke local. 
```dockerfile
version: '3.8'

services:
  gitlab-server:
    image: 'gitlab/gitlab-ce:17.5.2-ce.0'
    container_name: gitlab-server
    restart: unless-stopped
    ports:
      - "8088:8088"
      - "2224:22" # SSH
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://your-ip-address:8088' #bisa diisi juga dengan nama domain
        nginx['listen_port'] = 8088
        gitlab_rails['initial_root_password'] = 'your-super-secret-password'
        puma['worker_processes'] = 0 # Disable cluster mode to avoid more memory usage
        gitlab_rails['gitlab_ssh_host'] = 'http://your-ip-address'
        gitlab_rails['gitlab_shell_ssh_port'] = 2224
    volumes:
      - ./gitlab/config:/etc/gitlab
      - ./gitlab/logs:/var/log/gitlab
      - ./gitlab/data:/var/opt/gitlab
```

## Jalankan Docker
Untuk menjalankan docker compose, bisa menggunakan command berikut: \
`docker compose up -d`

Untuk pengecekan jika service sudah berjalan, gunakan command: `docker ps`. \
seharusnya akan muncul service docker dengan nama `gitlab-server`.

## Kesimpulan
Menginstall Gitlab Community Edition sangat cocok jika di sebuah organisasi memiliki server yang memadai dan ada orang khusus yang ditunjuk untuk mengelolanya.
Menggunakan docker juga baik untuk mengisolasi aplikasi dan kemudahan dalam installasi.

