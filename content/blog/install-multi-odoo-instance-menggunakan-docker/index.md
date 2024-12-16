+++
title = 'Docker Multi Instance Odoo'
description = 'tutorial dan cara install banyak odoo instance dengan docker'
date = 2024-09-29T19:03:00+07:00
draft = false
categories = ['Dev Ops']
tags = ['docker', 'odoo']
summary = "A brief step of installing odoo multi instance"
+++

Melanjutkan tulisan saya sebelumnya yaitu : 
[Install Odoo 17 Menggunakan Docker.](http://localhost:1313/posts/tutorial-install-odoo-17-menggunakan-docker/)

Kali ini kita akan mencoba install beberapa odoo instance dengan satu database yang sama. Saya berasumsi kalian sudah
sudah berhasil meng-install odoo menggunakan docker berdasarkan artikel saya sebelumnya.

## Matikan Servis Docker Compose
**Stop Servis Docker Compose**\
`docker compose -f docker-compose.yaml -f dev.yaml stop`

**Remove Docker Compose**\
`docker compose -f docker-compose.yaml -f dev.yaml down`

**Cek Service Docker Compose Terakhir**\
`docker compose ps -l`

## Konfigurasi docker-compose.yaml
Untuk aplikasi odoo yang baru, kita perlu membuat servis docker dan volume baru.

```dockerfile
version: "3.8"
services:
  odoo-demo:
    image: custom-odoo:17.0
    container_name: odoo-demo
    env_file: myenvfile.env
    depends_on:
      - postgres-demo
    volumes:
      - odoo1-data-demo:/var/lib/odoo
      - ./config/odoo/demo:/etc/odoo
      - ../../customize/odoo-demo/addons-customize:/mnt/addons-customize
      - ../../customize/odoo-demo/addons-thirdparty:/mnt/addons-thirdparty
    networks:
      - demo_odoo_network

  # odoo instance baru
  odoo-demo1:
    image: custom-odoo:17.0
    container_name: odoo-demo1
    env_file: myenvfile.env
    depends_on:
      - postgres-demo
    volumes:
      - odoo1-data-demo1:/var/lib/odoo
      - ./config/odoo/demo1:/etc/odoo
      - ../../customize/odoo-demo1/addons-customize:/mnt/addons-customize
      - ../../customize/odoo-demo1/addons-thirdparty:/mnt/addons-thirdparty
    networks:
      - demo_odoo_network
  # END odoo instance baru

  postgres-demo:
    image: postgres:15
    container_name: postgres-demo
    env_file: myenvfile.env
    volumes:
      - ./pg-data:/var/lib/pgsql/data/pgdata
      - ./config/postgresql/postgresql.conf:/etc/postgresql/postgresql.conf:ro
    networks:
      - demo_odoo_network

  # nginx-default:
  #   image: nginx:1.24.0
  #   container_name: nginx-default
  #   volumes:
  #     - ./config/nginx/nginx.conf:/etc/nginx/nginx.conf
  #   depends_on:
  #     - odoo17-default
  #   networks:
  #     - default_odoo_network

volumes:
  odoo1-data-demo:
  # tambahkan volume untuk odoo baru
  odoo1-data-demo1: 

networks:
  demo_odoo_network:
    name: demo_odoo_network
    driver: bridge
```

## Ubah Konfigurasi Extension
Selanjutnya, kita akan mengkonfigurasi file docker-compose extension (`dev.yam` & `prod.yaml`). Sebagai contoh, saya
hanya akan mengubah konfigurasi `dev.yaml` untuk menambahkan servis dan mengkonfigurasi port yang akan digunakan servis
`odoo-demo1`.

```dockerfile
version: "3.8"
services:
  odoo-demo:
    restart: unless-stopped
    ports:
      - "8069:8069"
  
  # servis odoo baru
  odoo-demo1:
    restart: unless-stopped
    ports:
      - "8070:8069"
  # END servis odoo baru
      
  postgres-demo:
    restart: unless-stopped
    ports:
      - "5433:5432"

  # nginx-demo:
  #   restart: no
  #   ports:
  #     - "81:80"
```

## Copy folder Konfigurasi Odoo
Sekarang kita copy folder `demo` yang berada di `docker-odoo\conf\odoo-demo\config\odoo\demo`, lalu paste dengan 
nama `demo1`. Untuk saat ini, samakan saja isi file tersebut dengan sebelumnya, atau bisa juga di ubah sesuai
dengan kebutuhan.

## Tambahkan Modul Kustomisasi Odoo
Lalu kita tambahkan modul kustomisasi Odoo di folder `customize`. Untuk saat ini, saya akan copy folder `odoo-demo`
lalu mengganti namanya menjadi `odoo-demo1`. Atau, bisa disesuaikan juga dengan kebutuhan kalian.

## Jalan Kembali Servis Docker
**Buat Servis Docker Compose Baru**\
``docker compose -f docker-compose.yaml -f dev.yaml up -d``

## Kesimpulan
Menginstall banyak odoo instance menggunakan satu database cocok untuk skenario berikut:
- Punya banyak project odoo
- Punya banyak project odoo dengan versi berbeda-beda
- Ingin mencoba odoo versi lain
- Tidak ingin menginstall Odoo langsung di mesin lokal kalian
- Membuat struktur folder lebih efisien

Terima Kasih!