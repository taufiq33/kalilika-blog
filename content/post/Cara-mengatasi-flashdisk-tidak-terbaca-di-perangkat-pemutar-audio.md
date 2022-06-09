---
title: "Cara Mengatasi Flashdisk Tidak Terbaca Di Perangkat Pemutar Audio"
date: 2022-05-29T16:47:52+07:00
draft: false
categories: "Solving Problem"
tags: ["flashdisk", "mp3"]
---

## Latar Belakang masalah
Sebenarnya saya jarang sekali memutar file lagu atau apapun itu yang berupa media suara menggunakan perangkat flashdisk. Tapi Ibu saya ini senang sekali lagu dangdut india dsb. Karena Beliau tidak begitu fasih tentang teknologi, akhirnya meminta saya untuk membelikan flashdisk dan mengisinya dengan file-file mp3 lagu untuk kemudian diputar di perangkat radio.

Kebetulan perangkat radio yang kami miliki punya fitur untuk memutar file mp3 dari flashdisk. Seperti ini perangkat radionya :

![perangkat Radio Ibu Saya](/images/radio.jpg)

Permasalahan muncul ketika saya meminjam flashdisk tersebut untuk install ulang Laptop Saya. Singkat cerita setelah install ulang, dan file-file mp3 sudah saya kembalikan ke flashdisk tsb. Ternyata Radio tidak dapat memutar file yang ada di flashdisk. Saya kemudian mencari beberapa solusi di internet, dan akhirnya berhasil. Saya berinisiatif menuliskannya disini.

## Solusi
Setelah _googling_ beberapa keyword, banyak yang menyarankan untuk format ulang, lalu rubah tipe filesystem ke FAT32 , format ulang tanpa checklist fast Format, dan masih banyak lagi.

Alhamdulillah saya menemukan solusi nya dari video Youtube dari channel Singgih Ikhsan19, dengan link => [ini](https://youtu.be/6P6FQkS2HkU)

Kesimpulannya adalah: 
1. Kebanyakan Perangkat pemutar audio dengan sumber media dari flashdisk, SD Card dsb itu hanya mengenali filesystem FAT32. Selain filesystem tsb, tidak akan bisa terbaca
2. Selain filesystem, partitionscheme dari perangkat flashdsik harus bertipe MBR (Master Boot Record). 

Dalam [video tersebut](https://youtu.be/6P6FQkS2HkU), flashdisk di format dengan menggunakan software rufus. Celakanya sejak lulus SMK saya tidak lagi menggunakan OS Windows dilaptop saya (Saya menggunakan OS GNU Linux). Sehingga saya pun mencari-cari alternatif software untuk format flashdisk. 

Saya menggunakan GNOMEDISK, lalu perintah terminal, namun keduanya sama-sama tidak berhasil. Akhirnya saya menggunakan laptop adik saya yang kebetulan menggunakan OS Windows 10. 

Sebelum nya, __Anda wajib backup file-file yang ada diflashdisk ya__.

Berikut cara perbaiki nya : 
1. Saya mengingatkan lagi untuk backup terlebih dahulu file yang ada di flashdisk Anda ya, karena proses ini akan menghapus seluruh file diflashdisk Anda. 
2. Pasang flashdisk anda ke laptop/komputer. Pastikan flashdisk terdeteksi.
3. Download software rufus. Anda dapat mendownloadnya disini => [Website Rufus](https://rufus.ie/en/)
4. Scroll kebawah sampai menemukan bagan Download. Pilih Link paling atas.
![websiterufus](/images/download_rufus_website.png)
5. Setelah terdownload, anda bisa langsung membuka softwarenya dengan cara klik 2x pada file yang anda download. (Software rufus tidak perlu instalasi)
6. Ini bagian terpenting. Pastikan anda mengatur bagian-bagian ini. 

__Device__ : _Pilih sesuai nama flashdisk Anda._

__BootSelection__ : _Pilih Non Bootable (Ini sangat berpengaruh, karena kasus saya flashdisk habis dipakai install ulang)_

__Partition Scheme__ : _MBR_

__Target system__ : _Bios UEFI_

__Volume Label__ : _isi bebas, ini akan menjadi nama flashdisk anda ketika berhasil diformat._

__File system__ : _FAT32(Default)_

__Cluster Size__ : _pada bagian ini anda pilih yang ada tulisan "(Default)" nya ya. Tiap flashdisk sepertinya punya nilai berbeda-beda. Kebetulan punya saya di angka 4096, sesuaikan saja dengan punya Anda masing-masing ya._

Untuk lebih jelasnya Anda bisa lihat di gambar ini :

![pengaturan_sebelum_format](/images/rufus_setting.jpg)


Pastikan sudah sesuai ya. 

Jika sudah Anda bisa klik tombol START.








