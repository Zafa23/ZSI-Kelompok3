# Jenkins Top On Docker 
- Pertama buat docker-compose lalu jalankan docker compose nya 
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

![1.png](Jenkins/1.png)
![2.png](Jenkins/2.png)

![3.png](Jenkins/3.png)
> perhatian: Jika kalian ga tau dimana posisi penimpanan password nya

![4.png](Jenkins/4.png)
>jika sudah mari kita atur untuk reverse proxy nya biar bisa diakses dengan domain yang kita mau

![5.png](Jenkins/5.png)
> Jika sudah ketemu password nya paste kesini 

![6.png](Jenkins/6.png) 

![7.png](Jenkins/7.png)
> setelah memilih plugin akan lanjut seperti ini dan kita harus menunggu sampai selesai

![8.png](Jenkins/8.png)
> Masukan domain yang udah kita buat di gateway vm

# Menghubungkan Jenkins Dengan Server
- Kemudian Konfigurasi Manage Credentials dan hubungkan Jenkins dengan VPS server dengan menambahkan ssh-keygen.
> public key dimasukan di dalam authorized keys.
private key diletakan pada user Credentials.

![9.png](Jenkins/9.png)
>Jika sudah pilih manage jenkins

![10.png](Jenkins/10.png)
![11.png](Jenkins/11.png)
![12.png](Jenkins/12.png)

- tambahkan ssh dan discord notifer, discord untuk kalau ada eror atau berhasil build ada info yang masuk di discord kita 

![13.png](Jenkins/13.png)

# Membuat File Jenkinsfile di frontend dan backend
![14.png](Jenkins/14.png)
> Selanjutnya buat Jenkinsfile, dan masukan seperti diatas ini

`frontend`
```
def branch = "main"
def repo = "https://github.com/Zafa23/wayshub-frontend.git"
def cred = "appserver"
def dir = "~/wayshub-frontend"
def server = "kel3@116.193.190.143"
def imagename = "wayshub-fe"
def dockerusername = "zsikelompok3"

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
                        docker container stop ${imagename}
                        docker container rm ${imagename}
                        docker run -d -p 3000:3000 --name="${imagename}"  ${imagename}:latest
                        docker container stop ${imagename}
                        docker container rm ${imagename}
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
				    docker image push ${dockerusername}/${imagename}:latest
				    docker image rm ${dockerusername}/${imagename}:latest
				    docker image rm ${imagename}:latest
				    exit
                    EOF
			"""
		        }
            }
        }
    }
}
```


`backend`
```
def branch = "main"
def repo = "https://github.com/myayangs/wayshub-backend.git"
def cred = "appserver"
def dir = "~/wayshub-backend"
def server = "kel1@103.13.206.133"
def imagename = "wayshub-be"
def dockerusername = "myyngstwn"

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
                        docker container stop ${imagename}
                        docker container rm ${imagename}
                        docker run -d -p 5000:5000 --name="${imagename}"  ${imagename}:latest
                        docker container stop ${imagename}
                        docker container rm ${imagename}
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
				    docker image push ${dockerusername}/${imagename}:latest
				    docker image rm ${dockerusername}/${imagename}:latest
				    docker image rm ${imagename}:latest
				    exit
                    EOF
			"""
		        }
            }
        }
    }
}
```

# Membuat pipeline 

- Kemudian pilih pipeline dan beri nama sesuai aplikasi.

- Masukkan ssh url github, branch dan dll. Hal ini dikarenakan setiap mengalami perubahan, penambahan pada Jenkinsfile harus di add, commit dan push ke akun github yang sudah di setting di server.

- Kemudian save. Jika Jenkinsfile sudah di tambahkan ke dalam repository. bisa langsung menekan build now. ketika di build bisa melihat error di bagian logs sesuai dengan stage yg di jalankan.


![15.png](Jenkins/15.png)
![16.png](Jenkins/16.png)
![17.png](Jenkins/17.png)
![18.png](Jenkins/18.png)
![19.png](Jenkins/19.png)
