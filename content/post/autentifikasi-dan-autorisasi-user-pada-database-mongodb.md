---
title: "Autentifikasi Dan Autorisasi User Pada Database Mongodb"
date: 2022-06-09T17:22:21+07:00
categories: "Tutorial"
tags: ["mongodb", "nosql"]
draft: false
---


# Latar Belakang Masalah 
Setelah install ulang kemarin, saya lupa untuk mencatat step by step membuat akun untuk database tertentu pada database MongoDB ini. Sebenarnya ini tidak terbatas pada akun root saja, tapi bisa juga akun dengan hak akses level lain. 

Secara default ketika kita menjalankan server MongoDB, (melalui perinth mongod), itu berjalan tanpa authentication. 

Saya menuliskan nya kembali disini.

# Prasyarat
Pastikan Anda sudah melakukan instalasi MongoDB. Bisadicek dengan perintah ```mongo --version``` . Oh iya saya menggunakan MongoDB versi 4.4.14. Untuk petunjuk instalasi bisa langsung menuju documentasi resmi nya di [https://www.mongodb.com/docs/v4.4/installation/](https://www.mongodb.com/docs/v4.4/installation/) .

```bash
taufiqh@taufiqh-Aspire-4739:~$ mongo --version
MongoDB shell version v4.4.14
Build Info: {
    "version": "4.4.14",
    "gitVersion": "0b0843af97c3ec9d2c0995152d96d2aad725aab7",
    "openSSLVersion": "OpenSSL 1.1.1f  31 Mar 2020",
    "modules": [],
    "allocator": "tcmalloc",
    "environment": {
        "distmod": "ubuntu2004",
        "distarch": "x86_64",
        "target_arch": "x86_64"
    }
}


```

# Langkah-Langkah
Untuk mengaktifkan fitur authentication pada MongoDB, Anda perlu merubah file config MongoDB, dengan menghapus karakter "#" pada security dan authorization. Dan nilai dari authorization adalah __enabled__. Secara default di Laptop saya (yang menggunakan OS GNU/Linux Zorin) itu ada di ```/etc/mongod/conf```.
```bash
# how the process runs
processManagement:
  timeZoneInfo: /usr/share/zoneinfo

#security:
#  authorization: enabled

#operationProfiling:


```
Tapi **Jangan dirubah dulu** .
Anda boleh merubahnya ketika user root sudah berhasil dibuat / Anda sudah membuat akun yang sudah diberi akses pada database tertentu.

Silahkan ikuti langkah berikut :

## Memastikan service MongoDB sudah berjalan.
Cara cek apakah sudah aktif atau belum adalah dengan mengetikkan perintah ```sudo systemctl status mongod``` .

```bash
taufiqh@taufiqh-Aspire-4739:~$ sudo systemctl status mongod
● mongod.service - MongoDB Database Server
     Loaded: loaded (/lib/systemd/system/mongod.service; disabled; vendor prese>
     Active: inactive (dead)
       Docs: https://docs.mongodb.org/manual

```

Nah kalau posisi seperti diatas berarti service MongoDB belum berjalan. Silahkan eksekusi perintah ```sudo systemctl start mongod``` . Lalu cek lagi statusnya. Pastikan sudah __Active__ .
```bash
taufiqh@taufiqh-Aspire-4739:~$ sudo systemctl start mongod
taufiqh@taufiqh-Aspire-4739:~$ sudo systemctl status mongod
● mongod.service - MongoDB Database Server
     Loaded: loaded (/lib/systemd/system/mongod.service; disabled; vendor prese>
     Active: active (running) since Thu 2022-06-09 16:12:39 WIB; 1s ago
       Docs: https://docs.mongodb.org/manual
   Main PID: 8540 (mongod)
     Memory: 39.9M
     CGroup: /system.slice/mongod.service
             └─8540 /usr/bin/mongod --config /etc/mongod.conf

Jun 09 16:12:39 taufiqh-Aspire-4739 systemd[1]: Started MongoDB Database Server.
lines 1-10/10 (END)

```
tekan 'q' untuk keluar dari mode status.

## Membuat Database, Collection dan beberapa document
untuk keperluan tutorial ini, silahkan buat _database_ ```perpustakaan``` , dengan _collection_ ```buku``` . Tambahkan 1-2 document juga ya.

```bash
taufiqh@taufiqh-Aspire-4739:~$ mongo --quiet
> use perpustakaan
switched to db perpustakaan
> db.buku.insertOne({
... "title": "Mata yang Enak Dipandang",
... "penulis": "Ahmad Tohari",
... "penerbit": "PT Gramedia Pustaka Utama",
... "tebal_halaman": "216",
... "isbn": "978-602-03005-9-7",
... });
{
	"acknowledged" : true,
	"insertedId" : ObjectId("62a1c0f5fed4d7df3d34314d")
}
> db.buku.find()
{ "_id" : ObjectId("62a1c0f5fed4d7df3d34314d"), "title" : "Mata yang Enak Dipandang", "penulis" : "Ahmad Tohari", "penerbit" : "PT Gramedia Pustaka Utama", "tebal_halaman" : "216", "isbn" : "978-602-03005-9-7" }
> db.buku.insertOne({
... "title": "Sapiens",
... "penulis":"Yuval Noah Harari",
... "penerbit": "PT Gramedia Pustaka Utama",
... "tebal_halaman": "526",
... "isbn":
... "978-602-424-416-3"});
{
	"acknowledged" : true,
	"insertedId" : ObjectId("62a1c221fed4d7df3d34314e")
}
> db.buku.find()
{ "_id" : ObjectId("62a1c0f5fed4d7df3d34314d"), "title" : "Mata yang Enak Dipandang", "penulis" : "Ahmad Tohari", "penerbit" : "PT Gramedia Pustaka Utama", "tebal_halaman" : "216", "isbn" : "978-602-03005-9-7" }
{ "_id" : ObjectId("62a1c221fed4d7df3d34314e"), "title" : "Sapiens", "penulis" : "Yuval Noah Harari", "penerbit" : "PT Gramedia Pustaka Utama", "tebal_halaman" : "526", "isbn" : "978-602-424-416-3" }
> 

```
```mongo --quiet``` adalah perintah untuk masuk ke mode shell MongoDB dengan menyembunyikan pesan log saat proses koneksi.

```use perpustakaan``` untuk membuat database dengan nama "perpustakaan".

```db.buku.insertOne``` ini sebenarnya command untuk menambahkan 1 document ke sebuah collection. Karena collection "buku" belum ada, otomatis akan dibuatkan. 

```db.buku.find()``` untuk menampilkan semua document yang ada pada collection "buku".

Bagi yang asing dengan istilah __document__ dan __collection__ , itu adalah nama lain dari __data/row__ dan __tabel__ pada Database Relational (DBMS).

## Membuat user khusus untuk mengakses database tertentu.
Nah setelah database perpustakaan tadi selesai dibuat, lanjut dengan membuat user khusus yang nantinya database perpustakaan ini hanya bisa diakses dengan user ini.
Pastikan shell mongoDB masih aktif ya, lalu ketikkan perintah :

```bash
> use admin
switched to db admin
> db.createUser({
... user: "admin-perpustakaan",
... pwd: "passwordkokgitu",
... roles:[
... {role: 'dbOwner', db: 'perpustakaan'}
... ]
... });
Successfully added user: {
	"user" : "admin-perpustakaan",
	"roles" : [
		{
			"role" : "dbOwner",
			"db" : "perpustakaan"
		}
	]
}
> 

```

```use admin``` ini perintah untuk mengakses database admin.

```bash
> db.createUser({
... user: "admin-perpustakaan",
... pwd: "passwordkokgitu",
... roles:[
... {role: 'dbOwner', db: 'perpustakaan'}
... ]
... });
```
```db.createUser``` diatas  berfungsi untuk membuat user baru.

```user``` adalah isian nama dari user yang ingin dibuat.

```pwd``` adalah isian kata sandi / password.

```roles``` adalah isian untuk role yang digunakan oleh user yang sedang dibuat serta database yang diberi akses.

berikut beberapa macam role yang tersedia : 
- read [membaca database]
- readWrite [membaca dan menulis database]
- dbAdmin [akses pengaturan skema, indexing dsb]
- userAdmin [akses untuk manajemen role]
- dbOwner [kombinasi dari ke empat role diatas]

role-role diatas pengaplikasiannya pada 1 database. Sementara jika Anda ingin memberi akses tsb yang konteksnya untuk semua database yang ada di MongoDB server, maka tambahkan prefix ```AnyDatabase``` diakhir yang berarti, itu berlaku untuk semua database.

- readAnyDatabase
- readWriteAnyDatabase
- dbAdminAnyDatabase
- userAdminAnyDatabase
- dbOwnerAnyDatabase

selain itu ada role ```root``` yang mana ini adalah role/hak akses tertinggi dan bisa melakukan apapun pada database manapun tergantung settingnya.

Anda bisa baca lengkap di dokumentasi resmi [https://www.mongodb.com/docs/v4.4/reference/built-in-roles/](https://www.mongodb.com/docs/v4.4/reference/built-in-roles/).

user 'admin-perpustaan' yang kita buat tadi memiliki role ```dbOwner```, yang berarti punya akses untuk baca, tulis, manajemen role, indexing dsb terhadap database 'perpustakaan'.

## Menggunakan akun yang sudah dibuat untuk mengakses database.

Nah inilah saatnya Anda mengaktifkan fitur authorization pada file config mongoDB. Buka file ```/etc/mongod/conf``` dengan teks editor kesayangan Anda. Lalu hapus karakter '#' pada bagian security dan authorization. Dan pastikan pada nilai authorization adalah __enabled__. Kurang lebih seperti ini hasilnya.

```bash
# how the process runs
processManagement:
  timeZoneInfo: /usr/share/zoneinfo

security:
  authorization: enabled

#operationProfiling:

```

Jangan lupa disave. Jika muncul error saat melakukan save file config tsb, maka pastikan Anda membuka file config tsb sebagai root. Jika di GNU/Linux tinggal menambahkan prefix sudo diawal command.

```bash
taufiqh@taufiqh-Aspire-4739:~$ sudo gedit /etc/mongod.conf
```

Langkah selanjutnya restart service mongoDB, dengan perintah :
```bash
sudo systemctl restart mongod
```

Cek dan pastikan service mongoDB tetap active (tidak ada error) dengan perintah :

```bash
sudo systemctl status mongod
```

```bash
taufiqh@taufiqh-Aspire-4739:~$ sudo systemctl restart mongod
taufiqh@taufiqh-Aspire-4739:~$ sudo systemctl status mongod
● mongod.service - MongoDB Database Server
     Loaded: loaded (/lib/systemd/system/mongod.service; disabled; vendor prese>
     Active: active (running) since Mon 2022-06-13 06:31:22 WIB; 7s ago
       Docs: https://docs.mongodb.org/manual
   Main PID: 4685 (mongod)
     Memory: 157.1M
     CGroup: /system.slice/mongod.service
             └─4685 /usr/bin/mongod --config /etc/mongod.conf

Jun 13 06:31:22 taufiqh-Aspire-4739 systemd[1]: Started MongoDB Database Server.
lines 1-10/10 (END)

```
tekan 'q' untuk keluar mode status tsb.

Setelah itu Anda bisa masuk mode Shell MongoDB dengan user yang sudah dibuat tadi.

```mongo --quiet -u [nama user]```

```bash
taufiqh@taufiqh-Aspire-4739:~$ mongo --quiet -u admin-perpustakaan
Enter password: 
> 
```

Untuk isian password silahkan diisi sesuai dengan data yang diberikan ketika membuat user.
Jika sudah muncul karakter ">" berarti login berhasil.

Sekarang mari kita coba menambah 1 buat document kedalam collection buku. 

```bash
> show databases;
perpustakaan  0.000GB
> use perpustakaan;
switched to db perpustakaan
> show collections
buku
> db.buku.find().pretty();
{
	"_id" : ObjectId("62a1c0f5fed4d7df3d34314d"),
	"title" : "Mata yang Enak Dipandang",
	"penulis" : "Ahmad Tohari",
	"penerbit" : "PT Gramedia Pustaka Utama",
	"tebal_halaman" : "216",
	"isbn" : "978-602-03005-9-7"
}
{
	"_id" : ObjectId("62a1c221fed4d7df3d34314e"),
	"title" : "Sapiens",
	"penulis" : "Yuval Noah Harari",
	"penerbit" : "PT Gramedia Pustaka Utama",
	"tebal_halaman" : "526",
	"isbn" : "978-602-424-416-3"
}
> db.buku.insertOne({
... title: 'Chairul Tanjung-Si Anak Singkong',
... penulis: ["Tjahja Gunawan Diredja", "Chairul Tanjung"],
... penerbit: "Buku Kompas",
... tebal_halaman: 402,
... isbn: "9789797096502"
... });
{
	"acknowledged" : true,
	"insertedId" : ObjectId("62a6866e414b354606399eaf")
}
> db.buku.find().pretty()
{
	"_id" : ObjectId("62a1c0f5fed4d7df3d34314d"),
	"title" : "Mata yang Enak Dipandang",
	"penulis" : "Ahmad Tohari",
	"penerbit" : "PT Gramedia Pustaka Utama",
	"tebal_halaman" : "216",
	"isbn" : "978-602-03005-9-7"
}
{
	"_id" : ObjectId("62a1c221fed4d7df3d34314e"),
	"title" : "Sapiens",
	"penulis" : "Yuval Noah Harari",
	"penerbit" : "PT Gramedia Pustaka Utama",
	"tebal_halaman" : "526",
	"isbn" : "978-602-424-416-3"
}
{
	"_id" : ObjectId("62a6866e414b354606399eaf"),
	"title" : "Chairul Tanjung-Si Anak Singkong",
	"penulis" : [
		"Tjahja Gunawan Diredja",
		"Chairul Tanjung"
	],
	"penerbit" : "Buku Kompas",
	"tebal_halaman" : 402,
	"isbn" : "9789797096502"
}
> exit
bye
```

Penambahan 1 document pada collection buku berhasil. Tentu saja, karena user 'admin-perpustakaan' punya akses utk melakukan itu.

Sekarang contoh saja ya Semisal admin-perpustakaan ingin membuat user khusus untuk pegawai baru yang baru memulai training. Sehingga user khusus ini hanya diberi akses read / membaca data saja.

Bagaimana caranya ?. Cukup ikuti intruksi seperti sebelumnya.

1. Nonaktifkan opsi authorization pada config file mongoDB (/etc/mongod.conf). Bisa dengan merubah jadi disabled / memberi karakter '#' di awal.
```bash
#security:
#   authorization: enabled
```

atau 

```bash
security:
   authorization: disabled
```

2. Jangan lupa restart service setelah merubah file config.

```sudo systemctl restart mongod```

3. Pastikan setelah restart service, service tetap aktif (tidak ada error). Cek dengan cara :

```sudo systemctl status mongod```

4. Nah sekarang masuk mode shell mongoDB tanpa auth, lalu buat user khusus untuk pegawai baru tsb.: 

```bash
taufiqh@taufiqh-Aspire-4739:~$ mongo --quiet
> use admin
switched to db admin
> db.createUser({
...   user: 'usertrainingperpus',
...   pwd: 'training',
...   roles: [{role: 'read', db: 'perpustakaan'}]
... })
Successfully added user: {
	"user" : "usertrainingperpus",
	"roles" : [
		{
			"role" : "read",
			"db" : "perpustakaan"
		}
	]
}
> exit

```
5. Aktifkan kembali opsi authorization pada config file mongodb. (/etc/mongod.conf)

```bash
security:
   authorization: enabled
```

6. restart service setelah merubah file config.

```sudo systemctl restart mongod```

7. Pastikan setelah restart service, service tetap aktif (tidak ada error). Cek dengan cara :

```sudo systemctl status mongod```


Sudah begitu saja. Anda bisa langsung tes untuk login ke mongo shell dengan user training tsb, dan coba tmbahkan 1 document ke collection buku, maka yang Anda dapatkan adalah error.

```bash
mongo -u usertrainingperpus
MongoDB shell version v4.4.14
Enter password: 
connecting to: mongodb://127.0.0.1:27017/?compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("2e0fe9e4-c07f-40dc-bc18-cc58f169cbd6") }
MongoDB server version: 4.4.14
> use perpustakaan;
switched to db perpustakaan
> db.buku.find().pretty();
{
	"_id" : ObjectId("62a1c0f5fed4d7df3d34314d"),
	"title" : "Mata yang Enak Dipandang",
	"penulis" : "Ahmad Tohari",
	"penerbit" : "PT Gramedia Pustaka Utama",
	"tebal_halaman" : "216",
	"isbn" : "978-602-03005-9-7"
}
{
	"_id" : ObjectId("62a1c221fed4d7df3d34314e"),
	"title" : "Sapiens",
	"penulis" : "Yuval Noah Harari",
	"penerbit" : "PT Gramedia Pustaka Utama",
	"tebal_halaman" : "526",
	"isbn" : "978-602-424-416-3"
}
{
	"_id" : ObjectId("62a6866e414b354606399eaf"),
	"title" : "Chairul Tanjung-Si Anak Singkong",
	"penulis" : [
		"Tjahja Gunawan Diredja",
		"Chairul Tanjung"
	],
	"penerbit" : "Buku Kompas",
	"tebal_halaman" : 402,
	"isbn" : "9789797096502"
}
> db.buku.insertOne({
... title: 'test',
... penulis: 'test',
... penerbit: 'test',
... tebal_halaman: 2,
... isbn: '12345',
... });
uncaught exception: WriteCommandError({
	"ok" : 0,
	"errmsg" : "not authorized on perpustakaan to execute command { insert: \"buku\", ordered: true, lsid: { id: UUID(\"2e0fe9e4-c07f-40dc-bc18-cc58f169cbd6\") }, $db: \"perpustakaan\" }",
	"code" : 13,
	"codeName" : "Unauthorized"
}) :
WriteCommandError({
	"ok" : 0,
	"errmsg" : "not authorized on perpustakaan to execute command { insert: \"buku\", ordered: true, lsid: { id: UUID(\"2e0fe9e4-c07f-40dc-bc18-cc58f169cbd6\") }, $db: \"perpustakaan\" }",
	"code" : 13,
	"codeName" : "Unauthorized"
})
WriteCommandError@src/mongo/shell/bulk_api.js:417:48
executeBatch@src/mongo/shell/bulk_api.js:915:23
Bulk/this.execute@src/mongo/shell/bulk_api.js:1163:21
DBCollection.prototype.insertOne@src/mongo/shell/crud_api.js:264:9
@(shell):1:1
> 

```

Anda bisa perhatikan ya, ketika usertrainingperpus membaca data, itu tidak terjadi error, tapi ketika akan menambah data,  langsung muncul error. Berarti setting hak akses sudah berhasil.

# Tambahan

Sebenarnya jika diperhatikan, setiap ingin menambah user harus menonaktifkan opsi authorization dulu, lalu setelah berhasil juga harus melakukan restart service mongoDB. Adakah cara yang lebih simpel??

Ada, yaitu dengan cara membuat user (juga), tapi user khusus yang memiliki roles ```root``` atau ```userAdminAnyDatabase``` . Sehingga ketika kita ingin menambah user , tidak perlu lagi rubah file config dan restart service.

Cukup Login sebagai user dengan roles ```root``` atau ```userAdminAnyDatabase``` , (dengan setting selalu authorization: enable). Buat user dan setting role, Lalu tidak perlu restart service lagi.


Sekian tutorial dari Saya, semoga bermanfaat.
