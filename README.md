# basic_docker
# Panduan Instalasi Docker di Linux

## Langkah 1: Hapus Dependencies Lama
Sebelum menginstal Docker Engine, hapus dependencies lama dengan menjalankan command berikut:
```
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

## Langkah 2: Instal Docker Engine
- Tambahkan kunci GPG resmi Docker dan daftarkan repository Docker ke repo Linux:
```
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
```
 *NB: Untuk Linux Mint, ganti "$VERSION_CODENAME" dengan "$UBUNTU_CODENAME"*
 - Install Docker CLI dengan perintah:
```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
- Install docker dengan versi tertentu
```
# cek versi tersedia
apt-cache madison docker-ce | awk '{ print $3 }'
```
- Cara install
```
VERSION_STRING=5:24.0.0-1~ubuntu.22.04~jammy
sudo apt-get install docker-ce=$VERSION_STRING docker-ce-cli=$VERSION_STRING containerd.io docker-buildx-plugin docker-compose-plugin
```

## Langkah 3: Cek apakah docker engine sudah terinstal
Running
``` 
sudo docker run hello-world
```
Jika muncul error docker: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
```
sudo systemctl enable docker
sudo systemctl start docker
systemctl status docker
sudo docker run hello-world
```
### Penjelasan Dasar
Images: Image adalah paket yang berisi semua yang diperlukan untuk menjalankan aplikasi, termasuk kode, runtime, library, variabel lingkungan, dan konfigurasi. Image bersifat read-only dan tidak dapat diubah setelah dibuat. Saat Anda menjalankan image, Docker membuat instance dari image tersebut yang disebut container.

Container: Container adalah instance dari image yang berjalan. Container berisi semua yang diperlukan untuk menjalankan aplikasi, termasuk sistem file, variabel lingkungan, dan kode. Container bersifat isolasi, artinya container satu tidak dapat mengganggu container lainnya, kecuali ada konfigurasi khusus yang memungkinkan interaksi antar-container.

Registry: Registry adalah tempat penyimpanan untuk image Docker. Docker Hub adalah registry publik yang paling populer untuk image Docker, tetapi Anda juga dapat menggunakan registry privat seperti Docker Trusted Registry (DTR) atau registry dari penyedia cloud seperti AWS ECR, Google Container Registry (GCR), atau Azure Container Registry (ACR). Registry memungkinkan Anda untuk berbagi image dengan orang lain atau menyimpan image untuk penggunaan internal.
### Command Dasar

#### List Images
```
docker images
```
#### Install Images
```
docker pull ${imagesname}:${option version}
contoh:
docker pull node:1.1.0
```
#### Delete Images
```
docker image rm ${name}:${verion}
contoh:
docker image rm mongo:2.1.0
```
#### List Container
```
docker container ls
atau
docker container ls --all
```
#### Install Container
```
docker container create --name ${optional name of container} ${name images}
# contoh
docker container create mongoserver mongo
```
#### Install Container agar bisa diakses dari luar
```
docker container create --name ${namecontainer} -p ${porteksternal}:${portinternal} ${nameimages}
# contoh setting di port 8080
docker container create --name mongoserver -p 8080:27017 mongo:4.1
```
#### Running Container
```
docker container start ${name container}
# contoh
docker container start app1
```
#### Stop Container
```
docker container stop ${name container}
# contoh
docker container stop app1
```

#### Stop Container
```
docker container rm ${name container}
# contoh
docker container rm app1
```
#### Cek inspect Container
```
docker container inspect ${containername}
```
#### Create Docker File
```
FROM golang:1.4.11 //install images
COPY main.go /app/main.go //pemindahan file dari local ke images
CMD ["go", "run", "/app/main.go"] //menjalankan app
```
Reference [dockerfile](https://docs.docker.com/engine/reference/builder/)

#### Cara Install Images By Dockerfile
```
docker build --tag ${appname}:${tag} ${locationfile}
# contoh
docker build --tag app1:1.0 niam/home
```
#### Cara Push Images Local Ke Hub
- buat repo dulu di docker hub setelah itu,
- docker push ${username}/${nameimages}:${tag} || docker push muhniam/app1:1.0
- samakan Images local dengan Images HUB = docker tag ${local image} ${online image} || docker tag app1:1.0 muhniam/app1:1.0
- lakukan lagi langkah ke2

#### Menghubungkan Container 1 dengan yang lain
Case: sebuah aplikasi yang membutuhkan database agar bisa untuk menyimpan data, dan memanfaatkan ENV.
```
#terdapat 2 buah image yaitu redis dan app1
#build container redis
docker container create --name redis -p 5678:5678 redis:5
#build container app1 dengan menggunakan container redis || -e untuk setting environment
docker container create --name app1 -p 8080:8080 -e ${key}:${val} app1:1.0 || docker container create --name app1 -p 8080:8080 -e REDIS_HOST:redis -e REDIS_PORT:5678 app1:1.0
```
#### Docker Network
Secara default, sebuah container dibangun secara terisolasi yang memungkinkannya beroperasi secara independen dan tidak langsung terhubung atau dipengaruhi oleh container lain. Maka dari itu digunakannya *Docker Network*. Sistem ini memungkinkan sebuah container dapat saling terhubung dan memanfaatkan dirinya satu sama lain, ibarat sebuah kolam yang diisi beberapa jenis ikan yang saling membutuhkan satu sama lain.
Command:
```
docker network help
docker network create ${name}
docker network connect ${networkname} ${containername}
```
#### Docker Compose
Docker Compose digunakan untuk mengelola aplikasi multi-container Docker. Ini memungkinkan Anda mendefinisikan aplikasi dengan menggunakan file YAML untuk mengonfigurasi layanan aplikasi, jaringan, dan volume yang terpisah. Dengan Docker Compose, Anda dapat dengan mudah menjalankan, menghentikan, dan menyiapkan aplikasi yang terdiri dari beberapa kontainer dengan satu perintah.
*Untuk lebih lengkapnya sudah saya buatkan file docker-compose.yml*
```
# running docker compose di backgroud dan generate container
docker compose up -d
# kill container dan delete container
docker compose down
# hanya start container tanpa nge build
docker compose start
# hanya stop container tanpa menghapus
docker compose stop
```
#### Docker Volume 
Docker volume digunakan untuk menyimpan data yang ingin dipertahankan atau dibagikan di antara container Docker. Volume memungkinkan data untuk tetap ada meskipun container dihapus, dihentikan, atau diperbarui. Mereka juga memungkinkan container berbagi data dengan cara yang aman dan efisien. Dengan menggunakan volume, Anda dapat memisahkan data dari container itu sendiri, yang berguna dalam pengelolaan data yang persisten dan pengembangan aplikasi yang terdistribusi.
```
# create docker volume
docker volume ${volumename} || docker volume redis_volume
# create container with volume [-v] for volume || ${volumename} bisa diganti dengan local diirectori
docker container create --name redis -p 5678:5678 -v ${volumename}:${db_folder} redis:${tag} || docker container create --name redis -p 5678:5678 -v redis_volume:/usr/local/etc/redis redis:5
# 
```

