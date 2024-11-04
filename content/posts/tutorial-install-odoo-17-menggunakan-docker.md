+++
title = 'Install Odoo 17 Menggunakan Docker'
date = 2024-09-29T18:41:16+07:00
draft = false
categories = ['web developer']
tags = ['odoo', 'docker']
+++


Tulisan ini akan menunjukkan cara meng-install Odoo dan Postgresql menggunakan Docker.
Sebelum mengikuti tutorial ini, penulis asumsikan bahwa kamu sudah meng-install Docker di operating sistem kalian.
Jika belum, silahkan ikuti [Tutorial Install Docker]([https://](https://docs.docker.com/engine/install/)) dari laman resminya.

Jika sudah, akan ada beberapa langkah yang kamu harus ikuti sampai Odoo dan Postgresql berhasil terinstall dan siap kamu gunakan.
1. Buat Struktur Direktori
2. Build Dockerfile untuk Odoo
3. Buat Docker Compose
4. Docker Compose Extension

### Struktur Direktori
![struktur-direktori](/images/01/struktur-direktori.jpg)

Silahkan buat Direktori dengan struktur seperti diatas.

### Dockerfile
Selanjutnya kita akan membuat Dockerfile untuk Odoo 17. Silahkan sesuaikan dependensi yang akan di install.
```Docker
FROM odoo:17.0
USER root
RUN apt-get -y update && apt-get install -y git xmlsec1 vim zip
RUN pip3 install --upgrade --no-cache-dir pip
RUN pip3 install --no-cache-dir astor
RUN pip3 install --no-cache-dir cachetools
RUN pip3 install --no-cache-dir openupgradelib
RUN pip3 install --no-cache-dir wheel
RUN pip3 install --no-cache-dir pandas
RUN pip3 install --no-cache-dir mdb-parser
RUN pip3 install --no-cache-dir holidays
RUN pip3 install --no-cache-dir bigjson
RUN pip3 install --no-cache-dir pymysql
RUN pip3 install --no-cache-dir formats
RUN pip3 install --no-cache-dir py3o.formats
RUN pip3 install --no-cache-dir py3o.template
# Password security
RUN pip3 install --no-cache-dir zxcvbn
```

Untuk build image dari dockerfile, gunakan command berikut: \
`docker build -t custom-odoo:17.0 .`

### Docker Compose
Silahkan buat file `docker-compose.yaml` dengan kode dibawah ini:
```Docker
version: "3.8"
services:
  odoo-demo:
    image: custom-odoo:17.0
    container_name: odoo-demo
    # env_file: myenvfile.env
    depends_on:
      - postgres-demo
    volumes:
      - odoo1-data-demo:/var/lib/odoo
      - ./config/odoo:/etc/odoo
      - ../../customize/extra-addons:/mnt/extra-addons
    networks:
      - demo_odoo_network

  postgres-demo:
    image: postgres:15
    container_name: postgres-demo
    env_file: myenvfile.env
    volumes:
      - ./pg-data:/var/lib/pgsql/data/pgdata
    networks:
      - demo_odoo_network


volumes:
  odoo1-data-demo:
    name: "odoo1-data-demo"

networks:
  demo_odoo_network:
    name: demo_odoo_network
    driver: bridge
```

### Docker Compose Extension
Biasanya, kita menjalankan docker di lokal dan juga server namun dengan konfigurasi yg berbeda pada file `docker-compose.yaml`. 
Dengan alasan kemudahan untuk konfigurasi, disini penulis membuat ekstensi untuk file konfigurasi `docker-compose.yaml` dengan nama `dev.yaml` dan `prod.yaml`. \
File `dev.yaml` digunakan untuk di lokal dan file `prod.yaml` digunakan untuk server. Isi dari kedua file tersebut mirip, tinggal disesuaikan saja dengan kebutuhan di lokal dan server. \
Dibawah ini penulis menggunakan file `dev.yaml` sebagai contoh. Silahkan menyesuaikan `prod.yaml` untuk kebutuhan server kalian.

```Dockerfile
version: "3.8"
services:
  odoo-demo:
    restart: unless-stopped
    ports:
      - "8069:8069"

  postgres-demo:
    ports:
      - "5433:5432"
    restart: unless-stopped
```

Jika sudah membuat `docker-compose.yaml` dan `dev.yaml`, silahkan jalankan command docker berikut:
`docker compose -f docker-compose.yaml -f dev.yaml up -d`

![struktur-direktori](/images/01/docker-ps.jpg)

Untuk cek apakah service docker sudah berjalan, silahkan gunakan command : `docker ps`.
Untuk cek log dari aplikasi yang sedang berjalan, bisa menggunakan command: `docker compose logs -f`. \
Atau, bisa juga melihat log berdasarkan container yang
sedang berjalan menggunakan command: `docker container logs -f ID_Container`. 

Sekarang saatnya untuk menjalankan service docker di browser. Jika tidak ada masalah, seharusnya kita bisa mengakses aplikasi odoo pada alamat `http://localhost:8069/`.
![struktur-direktori](/images/01/localhost.jpg)

### Epilog
Selamat, sekarang kalian sudah bisa meng-install Odoo dan Postgresql menggunakan docker. Dengan konfigurasi ini, teman-teman bisa meng-install multi-instance odoo dengan cara
meng-copy direktory `odoo-demo` dan merubah konfigurasinya sesuai dengan kebutuhan teman-teman.

Terimakasih.