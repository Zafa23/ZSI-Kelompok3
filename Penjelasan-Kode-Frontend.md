Frontend

def branch = "main": Mendefinisikan variabel branch dengan nilai "main". Ini mungkin digunakan untuk menyimpan nama cabang (branch) dari repository Git yang akan digunakan.

def repo = "[Frontend](https://github.com/Zafa23/wayshub-frontend.git)": Mendefinisikan variabel repo dengan nilai URL repository Git. Nilai ini menyimpan alamat repository dari proyek 'wayshub-frontend'.

def cred = "apserver": Mendefinisikan variabel cred dengan nilai "apserver". Ini mungkin digunakan untuk menyimpan kredensial (seperti nama pengguna dan kata sandi) untuk mengakses repository Git (jika diperlukan).

def dir = "~/wayshub-frontend": Mendefinisikan variabel dir dengan nilai direktori lokal "wayshub-frontend". Nilai ini menyimpan jalur direktori lokal di mana proyek 'wayshub-frontend' akan disalin saat diunduh dari repository.

def server = "zafar@103.82.92.255": Mendefinisikan variabel server dengan nilai "zafar@103.82.92.255". Ini mungkin digunakan untuk menyimpan alamat server tujuan di mana proyek akan diunggah atau dijalankan.

def imagename = "wayshub-fe": Mendefinisikan variabel imagename dengan nilai "wayshub-fe". Variabel ini kemungkinan akan digunakan untuk memberi nama gambar Docker saat proyek diimajinasi dan dikemas menjadi kontainer Docker.

def dockerusername = "zafarassidiq": Mendefinisikan variabel dockerusername dengan nilai "zafarassidiq". Ini kemungkinan digunakan sebagai nama pengguna Docker Hub atau registry Docker lainnya untuk mengunggah gambar Docker.

pipeline: Ini adalah blok utama untuk menginisialisasi sebuah Jenkins Pipeline.

agent any: Menggunakan agen Jenkins default untuk menjalankan pipeline di mana pun tersedia. Artinya, pipeline ini akan dieksekusi pada mesin agen apa pun yang tersedia di lingkungan Jenkins.

stages: Ini adalah blok di mana Anda mendefinisikan berbagai tahapan (stages) dalam pipeline.

stage('Pull From Repository'): Ini adalah nama dari tahap pertama dalam pipeline, yang disebut "Pull From Repository". Tahap ini akan menarik kode sumber dari repository Git.

steps: Ini adalah blok di mana langkah-langkah eksekusi pipeline didefinisikan.

sshagent([cred]): Ini adalah blok SSH Agent yang digunakan untuk mengelola kredensial SSH yang diperlukan untuk mengakses repository Git yang bersifat privat.

sh """...""": Ini adalah blok 'shell script' di mana perintah-perintah shell dieksekusi pada mesin agen Jenkins. Perintah-perintah ini akan dieksekusi dengan akses SSH melalui blok sshagent.

ssh -o StrictHostKeyChecking=no ${server} << EOF ... EOF: Perintah ssh digunakan untuk terhubung ke server tujuan (yang diwakili oleh variabel ${server}) menggunakan koneksi SSH. Opsi -o StrictHostKeyChecking=no menghilangkan pemeriksaan kunci host yang ketat, yang berarti tidak akan ada permintaan untuk mengonfirmasi atau menambahkan kunci host baru jika koneksi SSH pertama kali dibuat.

cd ${dir}: Perintah cd (change directory) digunakan untuk pindah ke direktori tujuan di server, yang diwakili oleh variabel ${dir}.

git remote add origin ${repo} || git remote set-url origin ${repo}: Perintah ini menambahkan atau mengatur URL remote dari repository Git (ditentukan oleh variabel ${repo}) dengan nama 'origin'. Jika 'origin' sudah ditambahkan sebelumnya, maka perintah 'git remote set-url' akan mengatur ulang URL remote.

git pull origin ${branch}: Perintah git pull digunakan untuk menarik (pull) kode dari cabang (branch) tertentu (ditentukan oleh variabel ${branch}) dari repository 'origin'.

exit: Perintah exit digunakan untuk keluar dari sesi SSH setelah perintah-perintah di atas dieksekusi.

Tahap 'Dockerize':
Tahap ini bertujuan untuk membangun (dockerize) proyek menggunakan Docker dan membuat sebuah gambar (image) Docker dari proyek tersebut.

docker build -t ${imagename}:latest .: Perintah docker build digunakan untuk membangun gambar Docker dari proyek yang ada di direktori ${dir} (yang merupakan direktori proyek setelah diunduh dari repository). Opsi -t digunakan untuk memberi nama pada gambar yang dibangun. Pada contoh ini, gambar tersebut dinamai dengan ${imagename}:latest.
Tahap 'Deploy Docker':
Tahap ini bertujuan untuk menerapkan (deploy) gambar Docker yang telah dibangun ke dalam kontainer dan menjalankannya di server tujuan.

docker stop ${imagename} || true: Perintah docker stop digunakan untuk menghentikan kontainer dengan nama ${imagename} jika kontainer tersebut sudah berjalan. Opsi || true digunakan untuk mengabaikan pesan error jika kontainer tidak berjalan.

docker rm ${imagename} || true: Perintah docker rm digunakan untuk menghapus kontainer dengan nama ${imagename} jika kontainer tersebut sudah dihentikan. Opsi || true digunakan untuk mengabaikan pesan error jika kontainer tidak ada atau tidak berjalan.

docker run -d -p 3000:3000 --name=${imagename} ${imagename}:latest: Perintah docker run digunakan untuk menjalankan kontainer baru dengan gambar ${imagename}:latest. Opsi -d digunakan untuk menjalankan kontainer dalam mode detasemen (background). Opsi -p 3000:3000 digunakan untuk meneruskan port 3000 dari kontainer ke port 3000 di server tujuan. Opsi --name=${imagename} digunakan untuk memberi nama pada kontainer yang dijalankan, sehingga kita bisa mengelola kontainer menggunakan nama tersebut.

Tahap 'Push to Docker Hub':
Tahap ini bertujuan untuk mengunggah gambar Docker yang telah dibangun ke Docker Hub atau registry Docker lainnya.

docker tag ${imagename}:latest ${dockerusername}/${imagename}:latest: Perintah docker tag digunakan untuk memberi tag (label) pada gambar Docker ${imagename}:latest dengan nama ${dockerusername}/${imagename}:latest. Ini mempersiapkan gambar untuk diunggah ke Docker Hub dengan nama pengguna ${dockerusername}.

docker login -u ${dockerusername} -p DOCKER_PASSWORD: Perintah docker login digunakan untuk masuk (login) ke Docker Hub atau registry Docker lainnya dengan menggunakan nama pengguna ${dockerusername} dan kata sandi ${DOCKER_PASSWORD}. Ini diperlukan untuk mengautentikasi diri sebelum mengunggah gambar.

docker push ${dockerusername}/${imagename}:latest: Perintah docker push digunakan untuk mengunggah gambar Docker ${imagename}:latest ke Docker Hub dengan nama pengguna ${dockerusername}.

Perlu dicatat bahwa pada tahap 'Push to Docker Hub', ada placeholder ${DOCKER_PASSWORD} yang tidak didefinisikan dalam kode ini. Seharusnya nilai ini diberikan sebagai variabel lingkungan atau secret yang diakses dalam pipeline agar kata sandi Docker Hub tetap rahasia dan aman.