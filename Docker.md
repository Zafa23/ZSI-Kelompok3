# Kelompok 3
Anggota
- Muhamad Abdul Zafar Assidiq
- Salman Alfarisi
- Muhammad Ilham 
***

# Docker 
Docker adalah platform perangkat lunak yang memungkinkan Anda untuk membuat, mengemas, dan menjalankan aplikasi secara konsisten di lingkungan yang terisolasi yang disebut "kontainer". Kontainer adalah unit terisolasi yang menjalankan perangkat lunak dan semua dependensinya, termasuk sistem operasi, perpustakaan, dan kode aplikasi. Docker memanfaatkan teknologi containerisasi yang sudah ada di dalam kernel Linux, seperti namespaces dan cgroups, untuk membuat dan mengelola kontainer.

## Requirements
- Buat 3 vm dengan spesifikasi sebagai berikut: 

| VM       | CPU     | RAM     | Storage |
|----------|---------|---------|---------|
| AppServer      | 2 Core  | 8 GB    | 100 GB  |
| Gateway        | 4 Core  | 16 GB   | 200 GB  |
| CI/CD          | 8 Core  | 32 GB   | 500 GB  |




![app-server.jpg](/_resources/app-server.jpg)

![gateway.jpg](_resources/gateway.jpg)

![b5ebc3415acda502dda7921a28d2845b.png](_resources/b5ebc3415acda502dda7921a28d2845b.png)

# Setup Docker 
Hal pertama setelah membuat vm app server ikuti petunjuk disini [Docker](https://docs.docker.com/engine/install/ubuntu/)


```
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
```
![1.jpg](_resources/1-3.jpg)

> Jalankan perintah diatas ini atau bisa copy text nya 

![3.jpg](_resources/3-1.jpg)
> Step ke dua masukan gpg key nya kalian bisa copy di bawah ini

```
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```
![4.jpg](_resources/4.jpg)
> Buat seperti ini untuk menyiapkan repository nya, setelah membuat repository lakukan `sudo apt-get update`

```
echo \
  > "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  > "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  > sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```


![5.jpg](_resources/5.jpg)
> Baru disini instalasi docker dengan perintah 

```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

![7.jpg](_resources/7.jpg)
> selanjutnya jalankan `docker -v` , kemudian setup untuk user root agar kita tidak perlu menggunakan sudo lagi saat menjalankan docker 

```
sudo usermod -aG docker (user)
```

![8.jpg](_resources/8.jpg)
> keterangan : perintah di atas ini adalah suatu perintah untuk mengizinkan user yang digunakan agar dapat menjalankan perintah docker tanpa menggunakan perintah sudo.

# Deploy Aplikasi On Top Docker

**Hal pertama harus mensiapkan aplikasi yang akan di deploy docker nya disini sudah disiapkan aada backend dan database nya**

```
git clone https://github.com/dumbwaysdev/wayshub-frontend
```
![1..jpg](_resources/1.jpg)
> Jika sudah lakukan cek dengan `ls` ini akan menampilkan perintah kalau aplikasi frontend sudah ada

![2..jpg](_resources/2.jpg)
> Sekarang intergrasikan si fronted agar tersambung oleh backend dengan masuk ke file api.js `cd wayshub-frontend/src/config/api.js` jika sudah kita akan membuat file Dockerfile di dalam wayshub-frontend

![3.jpg](_resources/3-2.jpg)
>Ini tampilan untuk di dalam api.js disini ubah dengan domain yang akan kita buat untuk backend

![4.jpg](_resources/4-1.jpg)
> Dan ini untuk scrript Dockerfile nya selanjutnya kita akan masuk konfiurasi database

## Database
**Sekarang buatlah di luar frontend file compose-mysql.yml**
![1.jpg](_resources/1-1.jpg)
![2..jpg](_resources/2-2.jpg)
> Jika sudah jalankan perintah `docker compose -f compose-mysql.yml up -d` 

![3..jpg](_resources/3-3.jpg)
> Selanjutnya masuk kedalam mysql menggunakan bash



**Backend**
![1.jpg](_resources/1-2.jpg)
> selaanjutnya clone backend nya ini link nya yang bisa di gunakan 

```
git clone https://github.com/dumbwaysdev/wayshub-backend
```



![2.jpg](_resources/2-1.jpg)
>selanjutnya sama seperti front end tapi disini yang di edit bukan src nya tapi langsung edit `nano config/config.json`, jika sudah lanjutkan membuat dockerfile di dalam wayshub-backend.

![3.jpg](_resources/3.jpg)
> Username dan psw di isi dengan username myasql yang udah dibuat di bash sebelumnya


![4.jpg](_resources/4-2.jpg)
> di sini sedikit beda sama di frontend disini tambahkan yang kalian butuhkan 

**Docker Compose** 
![5.jpg](_resources/5-1.jpg)
> Disini kita akan membuat docker-compose.yml setelah itu menjalankan untuk isinya seperti ini 

![6.jpg](_resources/6.jpg)
> Setelah membuat jaalan kan perintah ini.

**Docker Push Image**

![f5e2bdfdd317ab5ca430452c85b05c59.png](_resources/f5e2bdfdd317ab5ca430452c85b05c59.png)
>Masuk ke docker login jika diminta password dan username silakan masuk 

```
docker tag kel3-frontend zsikelompok3/wayshub-frontend
```


```
docker push zsikelompok3/wayshub-frontend
```

![4c31cc32bd6bef5ec82f781717d76e6f.png](_resources/4c31cc32bd6bef5ec82f781717d76e6f.png)
>lalu masukan perintah diatas 

# Konfigurasi Reverse Proxy
>Kemudian masuk ke`cd /etc/nginx` dan buat folder dengan nama apapun lalu buat konfigurasi untuk jenkins ,fronteb,backend setelah sudah semua pasangkan certbot 

![9e44c23e1e1f28284f66c7a3128b4418.png](_resources/9e44c23e1e1f28284f66c7a3128b4418.png)

![8d3a2e9a34d1561c52c26f11a056f0c2.png](_resources/8d3a2e9a34d1561c52c26f11a056f0c2.png)

![1325d230c52d43c89deb5fe6a2f10ce7.png](_resources/1325d230c52d43c89deb5fe6a2f10ce7.png)

# Hasil
![50dab7cb538a6023790c159219f3e923.png](_resources/50dab7cb538a6023790c159219f3e923.png)

![8ae7b456cc1d89d39b719a6df9a47a00.png](_resources/8ae7b456cc1d89d39b719a6df9a47a00.png)











