+++
title = 'Install Gitlab Runner'
date = '2025-02-10T08:38:29+07:00'
draft = false
description = 'Untuk menggunakan CI/CD pada gitlab, ada beberapa aplikasi untuk mendukung hal tersebut. Tapi, GitLab Runner adalah yang paling kompatibel dengan GitLab Community.'
categories= ['Dev Ops']
tags = ['gitlab','git']
+++

## Latar Belakang Masalah
Dalam proses deployment fitur ke user. Biasanya, ketika developer sudah menyelesaikan suatu fitur, maka terlebih dahulu akan
melakukan push source code fitur ke repository (misal: GitLab). Setelahnya, perubahan terbaru dari repository akan di pull di server untuk selanjutnya
dilakukan pengetesan dan lainnya.
Kira - kira alur kerjanya akan seperti ini:
1. Developer push branch baru ke remote repository.
2. Developer melakukan merge berisi source code fitur baru ke branch development/staging/production.
3. Developer login ke server, masuk ke directory project. Lalu melakukan pull source code project terbaru.
4. Terakhir, developer akan restart aplikasi.

Menggunakan CI/CD bisa menghilangkan langkah nomer 3 dan 4. Sebagai gantinya, kita bisa melakukan trigger langkah nomer 3 & 4
ketika ada perubahan pada branch repository (langkah nomer 2).

## Prerequisites
1. Untuk minimum requirement installasi gitlab runner bisa di cek di [halaman resmi](https://docs.gitlab.com/ee/ci/runners/hosted_runners/linux.html#machine-types-available-for-linux-x86-64) GitLab.
2. Memiliki akun GitLab. Jika ingin menggunakan GitLab CE, untuk installasi nya bisa cek [disini](https://rohimoz28.github.io/blog/install-gitlab-community/).

## Environemnt
Untuk installasi GitLab runner, saya akan menggunakan docker. Dan saya akan menggunakan GitLab CE sebagai akun GitLabnya.
Image docker GitLab Runner bisa cek [disini](https://hub.docker.com/r/gitlab/gitlab-runner). 

## Install GitLab Runner menggunakan docker
```dockerfile
version: '3.8'

services:
  gitlab-server:
    image: 'gitlab/gitlab-ce:17.5.2-ce.0'
    container_name: gitlab-server
    hostname: gitlab-server
    restart: unless-stopped
    ports:
      - "8088:8088"
      - "2224:22"
      - "443:443"
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://gitlab-server:8088'
        nginx['listen_port'] = 8088
        gitlab_rails['initial_root_password'] = 'Abcd@123456789'
        puma['worker_processes'] = 0
        gitlab_rails['gitlab_ssh_host'] = 'your-ip-address-or-domain'
        gitlab_rails['gitlab_shell_ssh_port'] = 2224
        gitlab_rails['gitlab_host'] = 'gitlab-server'
        gitlab_rails['gitlab_port'] = 8088
        gitlab_rails['gitlab_https'] = false
    networks:
      - gitlab-network
  
  # Docker GitLab Runner
  gitlab-runner:
    image: 'gitlab/gitlab-runner:alpine-v17.5.2'
    container_name: gitlab-runner
    hostname: gitlab-runner
    depends_on:
      - gitlab-server
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./gitlab-runner/gitlab_runner_config:/etc/gitlab-runner
    environment:
      - CI_SERVER=http://gitlab-server:8088
    networks:
      - gitlab-network

networks:
  gitlab-network:
    name: gitlab-network
```

**Note:** pastikan versi GitLab Runner dan GitLab CE kompatibel.