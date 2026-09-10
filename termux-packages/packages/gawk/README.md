# 

Mengatasi Bentrokan Variabel Makro `stack` (`fix-include-order.patch`)

Android mendefinisikan elemen bernama *stack* di dalam pustaka sistem (`sched.h`), sementara `gawk` menggunakan kata *stack* sebagai nama makro di `awk.h`. Hal ini memicu eror ketidakcocokan deklarasi.

Urutan include berkas harus dibalik agar tidak bentrok:
1. Buka berkas `gawkmisc.c` dengan text editor Anda (misal `nano gawkmisc.c`).
2. Cari baris `#include "awk.h"` di bagian paling atas (sekitar baris 24-27). Hapus atau beri tanda komentar pada baris tersebut.
3. Buka berkas `posix/gawkmisc.c` (`nano posix/gawkmisc.c`).
4. Cari baris #include <spawn.h>. Tambahkan baris #include "awk.h" tepat di bawahnya.

<br>

---

<br>

Menyesuaikan Ekstensi Stack (`stack_index.patch`)

Modifikasi ini diperlukan karena kata `index` adalah fungsi bawaan yang terkadang memicu ambiguitas nama variabel pada arsitektur tertentu.
1. Buka berkas `extension/stack.c` (`nano extension/stack.c`).
2. Cari variabel bernama `index`.
3. Ubah semua kata `index` di dalam berkas tersebut menjadi `stack_index` (misalnya: `static int index = -1;` diubah menjadi `static int stack_index = -1;`).

<br>

---

<br>

Menonaktifkan Fungsi Pengguna Akun (`no_pw_gecos.patch`)

Sistem operasi Android tidak memiliki skema manajemen pengguna `/etc/passwd` konvensional layaknya Linux PC. Oleh sebab itu, fungsi pencarian `getpwent()` harus diisolasi agar tidak merusak kompilasi.
1. Buka berkas `awklib/eg/lib/pwcat.c`.
2. Kurung isi fungsi utama di dalam fungsi `main` menggunakan direktif pra-prosesor `#ifndef __ANDROID__`:
```c
int main(int argc, char **argv)
{
#ifndef __ANDROID__
    struct passwd *p;
    while ((p = getpwent()) != NULL)
        ...
    endpwent();
#endif
    return 0;
}

```

<br>

---

<br>

Jalankan Kompilasi Tanpa Semua Core

Setelah modifikasi file di atas selesai disesuaikan dengan lingkungan lokal Termux, Anda bisa langsung mengonfigurasi dan mengompilasi program dengan aman menggunakan jumlah core yang dibatasi (misal menggunakan 2 core):
```bash
# 1. Konfigurasi awal dengan mematikan Persistent Memory sesuai skrip asli Anda
./configure --prefix=$PREFIX --disable-pma

# 2. Mulai proses kompilasi dengan membatasi hanya menggunakan 2 core CPU
make -j2

# 3. Pasang gawk ke dalam sistem Termux
make install

```

<br>

---

<br>

**Secara resmi tetap tidak bisa** menggunakan perintah `./build-package.sh` langsung di Termux HP. Sistem manajemen paket Termux dirancang khusus untuk mendeteksi lingkungan perangkat, dan skrip pembangun tersebut akan otomatis memblokir proses jika mendeteksi bahwa ia dijalankan langsung di atas perangkat Android (bukan di Linux PC/Docker).

Namun, jika tujuan Anda adalah **otomatisasi penuh agar tidak perlu mengedit kode satu per satu secara manual**, Anda bisa meniru cara kerja Docker Termux secara lokal. Kita bisa memanfaatkan utilitas `patch` bawaan Linux untuk memasang semua berkas `.patch` tadi secara otomatis dalam satu detik.

Berikut adalah alur otomatisasi penuh langsung di Termux HP Anda:

Langkah 1: Persiapan Alat & Source Code

Pastikan Anda berada di direktori utama Termux dan unduh semua kebutuhan kompilasi beserta alat `patch`:
```bash
pkg install clang make pkg-config libgmp libmpfr readline tar curl patch micro

```

Unduh kode sumber gawk dan ekstrak:
```bash
curl -fsSLO https://mirrors.kernel.org/gnu/gawk/gawk-5.4.1.tar.xz && sleep 0.5 \
tar -xf gawk-5.4.1.tar.xz && sleep 0.5 \
cd gawk-5.4.1

```


```bash
# 1. Unduh patch urutan include
curl -fsSLO https://github.com/eucalypsih/ey_tp/raw/main/termux-packages/packages/gawk/fix-include-order.patch && sleep 0.5 && patch -p1 < fix-include-order.patch

# 2. Unduh patch untuk locale Android
curl -fsSLO https://github.com/eucalypsih/ey_tp/raw/main/termux-packages/packages/gawk/fix-locale.patch && sleep 0.5 && patch -p1 < fix-locale.patch

# 3. Unduh patch untuk penyesuaian index stack
curl -fsSLO https://github.com/eucalypsih/ey_tp/raw/main/termux-packages/packages/gawk/stack_index.patch && sleep 0.5 && patch -p1 < stack_index.patch

# 4. Unduh patch untuk menonaktifkan pencarian akun user (getpwent)
curl -fsSLO https://github.com/eucalypsih/ey_tp/raw/main/termux-packages/packages/gawk/no_pw_gecos.patch && sleep 0.5 && patch -p1 < no_pw_gecos.patch


curl -fsSLO https://github.com/eucalypsih/ey_tp/raw/main/termux-packages/packages/gawk/fix-fwrite-unlocked.patch && sleep 0.5 && patch -p1 < fix-fwrite-unlocked.patch

curl -fsSLO https://github.com/termux/termux-packages/blob/master/packages/gawk/io.c.patch

# 2. Ubah teks @TERMUX_PREFIX@ di dalam file patch menjadi variabel lingkungan asli Termux ($PREFIX)
sed -i "s|@TERMUX_PREFIX@|$PREFIX|g" io.c.patch

```

```
mkdir a b
cp awk.h.orig a/awk.h
cp awk.h b/awk.h
diff -u a/awk.h b/awk.h > fix-fwrite-unlocked.patch
```
```bash
diff -u awk.h.orig awk.h > fix-fwrite-unlocked.patch

```






















<br>
