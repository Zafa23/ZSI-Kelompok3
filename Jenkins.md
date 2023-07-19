![1.png](../_resources/1-1.png)
>Pertama-tama buat file bernama docker-compose.yml yang isi nya seperti di gambar atau bisa pakai yang telah saya buat, jika sudah jalankan perintah `docker compose up -d`

```
version: '3.8'
services:
   jenkins:
      image: jenkins/jenkins:lts-jdk11
      container_name: jenkins
      restart: always
      privileged: true
      user: root
      ports:
         - 8080:8080
         - 50000:50000 
      volumes:
         - ~/jenkins:/var/jenkins_home
```

![2.png](../_resources/2-1.png)
> Selanjutnya membuat reverse proxy dan memberi certbot pertama buatlah file di gateway `cd /etc/nginx`dengan type configurasi atau kalian bisa salin saja `sudo nano jenkins.conf` jika sudah ikuti langkah langkah pasang cert bot lewat documentasion ini [Certbot](https://certbot.eff.org/instructions?ws=nginx&os=ubuntufocal)


![3.png](../_resources/3-1.png)
> Jika kalian tidak bisa menemukan isi dari file initialAdminPassword kalian bisa mencari nya dengan memasukan rumus find disini sebagai contoh saja `sudo find . -name "initialAdminPassword" -type f` jika sudah kalian akan mendapatkan posisi dimana file initialAdminPassword diletakan setelah itu kalian bisa menggunakan rumus cat dan menggabungkan dengan hasil dari rumus find nya yang tadi


![4.png](../_resources/4-1.png)
>Disini kalian cukup ceklis sebelumnya yaitu yang diutamakan ssh-agent,git,pipeline jika telanjur bisa di dalam setelah instalasion selesai


![5.png](../_resources/5-1.png)
> Jika sudah kalian cukup buat akun baru selayaknya kalian bikin akun pada umumnya 


![6.png](../_resources/6-1.png)
> masukan konfigurasi domain yang tadi sudah dibuat sebelumnya 


![7.png](../_resources/7-1.png)
> Disini kita akan menghubungkan antara jenkins dengan appserver menggunakan id_rsa private kalian atau kalian bisa mengikuti step ini
>  `cat .ssh/id_rsa` selanjutnya kalian tempel di dalam sini dan isi seperti di gambar 


![8.png](../_resources/8-1.png)
> disini karna ssh-agent sudah terceklis diawal kita cukup memasukan discord agar ketika kita membuild nanti frontend atau backend bisa di masukan di grup discord kita dan akan ada notif berhasil atau gagal nya 



![9.png](../_resources/9-1.png)
> Disini kita masuk ketahap membuat file frontend nanti akan ku jelaskan isi dari si file ini kalian bisa pakai kode ini salin kode nya di bawah  

```
def branch = "main"
def repo = "https://github.com/Zafa23/wayshub-frontend.git"
def cred = "apserver"
def dir = "~/wayshub-frontend"
def server = "zafar@103.82.92.255"
def imagename = "wayshub-fe"
def dockerusername = "zafarassidiq"

pipeline {
    agent any

    stages {
        stage('Pull From Repository') {
            steps {
                sshagent([cred]) {
                    sh """ssh -o StrictHostKeyChecking=no ${server} << EOF
                        cd ${dir}
                        git remote add origin ${repo} || git remote set-url origin ${repo}
                        git pull origin ${branch}
                        exit
                        EOF
                    """
                }
            }
        }

        stage('Dockerize') {
            steps {
                sshagent([cred]) {
                    sh """ssh -o StrictHostKeyChecking=no ${server} << EOF
                        cd ${dir}
                        docker build -t ${imagename}:latest .
                        exit
                        EOF
                    """
                }
            }
        }

        stage('Deploy Docker') {
            steps {
                sshagent([cred]) {
                    sh """ssh -o StrictHostKeyChecking=no ${server} << EOF
                        cd ${dir}
                        docker stop ${imagename} || true
                        docker rm ${imagename} || true
                        docker run -d -p 3000:3000 --name=${imagename} ${imagename}:latest
                        exit
                        EOF
                    """
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                sshagent([cred]) {
                    sh """ssh -o StrictHostKeyChecking=no ${server} << EOF
                        docker tag ${imagename}:latest ${dockerusername}/${imagename}:latest
                        docker login -u ${dockerusername} -p DOCKER_PASSWORD
                        docker push ${dockerusername}/${imagename}:latest
                        exit
                        EOF
                    """
                }
            }
        }
    }
}
```

>Note : Disini saya membuat juga versi penjelasan dari kode yang saya buat berikut link nya [Frontend](https://github.com/Zafa23/ZSI-Kelompok3/blob/main/Penjelasan-Kode-Frontend.md)


![10.png](../_resources/10-1.png)
> Disini buatlah pipeline berjudul wayshub-fe atau kalian mau itu bebas sesuai penamaan aplikasi dan file saja yang saya ambil, dikarenakan saya mau mengotomasikan atau pengujian dari aplikasi wayshub pada folder frontend maka saya namakan wayshub-fe



![11.png](../_resources/11-1.png)
> Disini masukan seperti di gambar saja 




![12.png](../_resources/12-1.png)
> jika sudah sekarang build now dan tunggu sampai selesai jika ada keslahan kalian bisa arahkan cursor kalian ke bagian yang failed itu dan nanti akan muncul logs dan kalian pencet saja logs nya maka akan muncul perintah eror nya dan klian bisa perbaiki 




![13.png](../_resources/13-1.png)
> Dan sekarang buatlah file di backend dengan sama seperti di frontend yaitu `nano Jenkinsfile` dan masukan kode ini atau kalian bisa mencari sesuai tugas kalian 

```
def branch = "main"
def repo = "https://github.com/Zafa23/wayshub-backend.git"
def cred = "appserver"
def dir = "~/wayshub-backend"
def server = "zafar@103.82.92.255"
def imagename = "wayshub-be"
def dockerusername = "zafarassidiq"

pipeline {
    agent any

    stages {
        stage('Pull From Repository') {
            steps {
                sshagent([cred]) {
                    sh """ssh -o StrictHostKeyChecking=no ${server} << EOF
                        cd ${dir}
                        git remote add origin ${repo} || git remote set-url origin ${repo}
                        git pull origin ${branch}
                        exit
                        EOF
                    """
                }
            }
        }

        stage('Dockerize') {
            steps {
                sshagent([cred]) {
                    sh """ssh -o StrictHostKeyChecking=no ${server} << EOF
                        cd ${dir}
                        docker build -t ${imagename}:latest .
                        exit
                        EOF
                    """
                }
            }
        }

        stage('Deploy Docker') {
            steps {
                sshagent([cred]) {
                    sh """ssh -o StrictHostKeyChecking=no ${server} << EOF
                        cd ${dir}
                        docker stop ${imagename} || true
                        docker rm ${imagename} || true
                        docker run -d -p 5000:5000 --name=${imagename} ${imagename}:latest
                        exit
                        EOF
                    """
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                sshagent([cred]) {
                    sh """ssh -o StrictHostKeyChecking=no ${server} << EOF
                        docker tag ${imagename}:latest ${dockerusername}/${imagename}:latest
                        docker login -u ${dockerusername} -p DOCKER_PASSWORD
                        docker push ${dockerusername}/${imagename}:latest
                        exit
                        EOF
                    """
                }
            }
        }
    }
}
```
> Note : Untuk penjelasan hampir sama kaya frontend ya karna disini perintah nya sama yaitu 

- [x] Pull dari repository kita
- [x] Dockerize aplikasi kita
- [x]  Deploy aplikasi on top Docker
- [x] Push ke Docker Hu  

![14.png](../_resources/14-1.png)
>jika sudah semua lakukan sama yaitu build 

![15.png](../_resources/15-1.png)
>lalu periksa apakah sudah ada folder kita di dockerhub kita


![17.png](../_resources/17-1.png)
> Dan jangan lupa untuk buat auto triger, jika ada sesuatu atau peristiwa yang memungkinkan tindakan otomatis atau skrip untuk dijalankan secara otomatis ketika ada peristiwa tertentu terjadi dalam repositori GitHub. 
