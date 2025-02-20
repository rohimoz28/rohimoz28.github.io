+++
title = 'Konfigurasi Dasar CI/CD Gitlab'
date = '2025-02-20T15:32:40+07:00'
draft = true
description = 'Berikut ini adalah konfigurasi dasar file gitlab-ci.yml yang digunakan untuk CI/CD pada Gitlab.'
categories= ['Gitlab', 'Dev Ops', 'CI/CD']
tags = ['docker', 'odoo']
+++
## Setup worker pada GitLab
Sekarang saatnya kita registrasi runner pada gitlab.

### Register Runner
* Silahkan ke menu _Settings → CI/CD → Runners → New Project Runner_
* Isikan nama tags
* Ceklis _run untagged jobs_
* Klik _Create Runner_
* Lalu nanti akan muncul halaman seperti dibawah ini
![token-register-gitlab-runner](images/create-new-runner2.png)

### Register Token Pada Container GitLab Runner
* Copy token yang ada di kolom _Step 1_
* Masuk ke dalam container gitlab runner
* Paste token yang tadi di copy
![register-token-inside-container](images/register-runner-inside-container.png)