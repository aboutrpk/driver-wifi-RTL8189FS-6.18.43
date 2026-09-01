# Driver WiFi RTL8189FS untuk Kernel 6.18.43

Driver ini dikompilasi khusus untuk penggunaan pada **STB1 (SixZuper-Studio)** dengan kernel versi .

## Persyaratan

- Kernel headers untuk versi yang sesuai:
  
- Arsitektur prosesor: ARMv8 (aarch64)

## Cara Instalasi

1. Kloning repositori ini:
   

2. Kompilasi driver:
   

3. Instal modul ke sistem:
   

4. Muat driver secara manual:
   

## Verifikasi

Setelah menginstal, verifikasi apakah driver sudah dimuat dengan benar:
8189fs               1536000  0
cfg80211              872448  1 8189fs

Jika semuanya berhasil, antarmuka  seharusnya sudah muncul.

## Catatan Penting

- Pastikan versi kernel sudah sesuai dan header yang diperlukan sudah terpasang.
- Jika terjadi kesalahan kompilasi, periksa log untuk detailnya.
- Selalu backup konfigurasi jaringan sebelum mengganti driver WiFi.

## Lisensi

Driver ini mengikuti lisensi dari proyek aslinya dan dibagikan untuk tujuan edukasi dan penggunaan pribadi dalam ekosistem SixZuper-Labs By Mr. R.
