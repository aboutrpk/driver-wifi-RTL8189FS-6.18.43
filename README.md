# Driver WiFi RTL8189FS untuk Kernel 6.18.43

Driver ini dikompilasi khusus untuk penggunaan pada **STB1 (SixZuper-Studio)** dengan kernel versi `6.18.43-ophub`.

## Persyaratan

- Kernel headers untuk versi yang sesuai:
  ```bash
  sudo apt install linux-headers-6.18.43-ophub build-essential git curl xz-utils dkms make
  ```
- Arsitektur prosesor: ARMv8 (aarch64)
- Akses root untuk instalasi
- Koneksi internet untuk mengunduh toolchain

## Prasyarat: Unduh ARM GNU Toolchain

Driver ini memerlukan **ARM GNU Toolchain versi 15.3** yang sesuai dengan OPhub kernel. Jika belum terpasang:

```bash
TOOLCHAIN_NAME="arm-gnu-toolchain-15.3.rel1-aarch64-aarch64-none-linux-gnu"
TOOLCHAIN_URL="https://github.com/ophub/kernel/releases/download/dev/${TOOLCHAIN_NAME}.tar.xz"
curl --fail --location --retry 3 --output "${TOOLCHAIN_NAME}.tar.xz" "${TOOLCHAIN_URL}"
sudo install -d -m 755 /usr/local/toolchain
sudo tar -xJf "${TOOLCHAIN_NAME}.tar.xz" -C /usr/local/toolchain
```

Setelah berhasil, toolchain akan tersedia di `/usr/local/toolchain/arm-gnu-toolchain-15.3.rel1-aarch64-aarch64-none-linux-gnu`.

## Cara Instalasi Otomatis (Direkomendasikan)

Gunakan script `install.sh` yang disertakan. Script ini akan otomatis:
1. Verifikasi toolchain dan header kernel
2. Unduh toolchain jika belum tersedia
3. Kompilasi driver secara lokal menggunakan DKMS
4. Instal modul ke sistem kernel saat ini
5. Konfigurasi agar modul dimuat otomatis saat boot

```bash
chmod +x install.sh
sudo ./install.sh
```

> ⚠️ Pastikan Anda sudah login sebagai root atau menggunakan `sudo`.

## Cara Instalasi Manual (Alternatif)

Jika ingin melakukan kompilasi manual:

### Langkah 1: Siapkan Lingkungan Kompilasi

```bash
export TOOLCHAIN_ROOT="/usr/local/toolchain/arm-gnu-toolchain-15.3.rel1-aarch64-aarch64-none-linux-gnu"
export KSRC="/lib/modules/$(uname -r)/build"
```

### Langkah 2: Kompilasi Driver

```bash
cd driver
make ARCH=arm64 CC="${TOOLCHAIN_ROOT}/bin/aarch64-none-linux-gnu-gcc" KSRC="${KSRC}"
```

Setelah kompilasi selesai, file modular akan berada di `driver/8189fs.ko`.

### Langkah 3: Instalasi Modul secara Manual

```bash
sudo mkdir -p /lib/modules/$(uname -r)/kernel/drivers/net/wireless/realtek/
sudo cp driver/8189fs.ko /lib/modules/$(uname -r)/kernel/drivers/net/wireless/realtek/
sudo depmod -a && sudo modprobe 8189fs
```

### Langkah 4: Konfigurasi Modul Autoload

Agar driver dimuat otomatis saat boot:

```bash
sudo sh -c "echo 8189fs > /etc/modules-load.d/8189fs.conf"
sudo sh -c "echo \"options 8189fs rtw_power_mgnt=0 rtw_enusbss=0\" > /etc/modprobe.d/8189fs.conf"
```

## Verifikasi Instalasi

Setelah instalasi selesai, lakukan verifikasi dengan perintah berikut:

### Cek Modul Kernel

```bash
lsmod | grep 8189fs
```

Jika driver berhasil dimuat, Anda akan melihat baris seperti:
```
8189fs               xxx  0
```

### Cek Antarmuka Jaringan

```bash
ip link show wlan0
```

Jika berhasil, antarmuka `wlan0` akan muncul.

### Cek Status DKMS

```bash
dkms status | grep rtl8189fs
```

Output yang diharapkan:
```
rtl8189fs/5.7.9: installed on $(uname -r)
```

### Cek Deteksi Hardware

```bash
dmesg | grep -i 8189fs | tail -n 10
```

Ini akan menampilkan pesan kernel terkait inisialisasi perangkat WiFi.

## Konfigurasi Jaringan

Setelah `wlan0` tersedia, sambungkan ke jaringan WiFi:

```bash
# Sambungkan ke jaringan (tanpa password)
sudo iw dev wlan0 connect "NAMA_SSID"

# Atau dengan password
sudo iw dev wlan0 connect "NAMA_SSID" password "PASSWORD"
```

Untuk konfigurasi persisten, gunakan `netplan` atau `NetworkManager`.

## Catatan Penting

- ✅ Driver ini sudah **terverifikasi bekerja** di STB1 dengan kernel `6.18.43-ophub`
- ✅ Hasil keberhasilan: `wlan0` muncul, terhubung ke internet, IP: `192.168.1.9`
- ⚠️ Gunakan **toolchain khusus OPhub kernel** (v15.3) agar kompatibel
- ⚠️ Jangan lakukan `apt full-upgrade` tanpa rencana rebuild driver
- 🛠️ Untuk rebuild setelah update kernel, jalankan kembali `./install.sh`

## Struktur File

```
.
├── README.md              # Dokumentasi ini
├── install.sh             # Script instalasi otomatis
├── dkms.conf              # Konfigurasi DKMS
├── .gitignore
└── driver/                # Kode sumber driver
    ├── Makefile
    ├── Kconfig
    └── (file sumber lengkap...)
```

## Lisensi

Driver ini mengikuti lisensi dari proyek aslinya dan dibagikan untuk tujuan edukasi dan penggunaan pribadi dalam ekosistem SixZuper-Labs By Mr. R.
