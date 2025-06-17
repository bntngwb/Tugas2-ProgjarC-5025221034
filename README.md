🤖 Tugas 2: Time Server TCP dengan Multithreading

Repositori ini berisi implementasi dari sebuah time server sederhana menggunakan Python, yang dibuat sebagai bagian dari tugas mata kuliah Praktikum Pemrograman Jaringan (Kelas C).

Disusun oleh:

Nama: Bintang Wibi Hanoraga
NRP: 5025221034

---

##  Deskripsi Tugas

Tugas ini merupakan sebuah peladen (server) TCP yang berjalan di **port 45000**. Tujuan utamanya adalah untuk memenuhi permintaan informasi waktu dari berbagai klien secara serentak (*concurrently*) dengan memanfaatkan desain *multithreading*. Setiap entitas klien yang tersambung akan diurus oleh *thread* yang berbeda, memastikan bahwa peladen tetap responsif dan tidak terhambat oleh satu koneksi pun.

Peladen ini menerapkan protokol berbasis teks yang lugas:
1.  Klien mengirimkan `TIME` guna meminta informasi waktu terkini.
2.  Peladen memberikan respons dalam format `JAM hh:mm:ss`.
3.  Klien mengirimkan `QUIT` untuk mengakhiri sesi.

---

## Rancangan & Struktur

Program ini dibangun dengan dua kelas pokok untuk mencapai paralelisme:

-   **`Server` (Kelas)**: Berfungsi sebagai pemerhati (listener) utama. Tugasnya tunggal: memantau sambungan masuk pada port 45000. Ketika sebuah sambungan diterima, kelas ini tidak memprosesnya, melainkan segera mendelegasikannya dengan menciptakan instans baru dari `ProcessTheClient`.
-   **`ProcessTheClient` (Kelas)**: Merupakan *thread* pekerja. Setiap contoh dari kelas ini beroperasi dalam *thread*-nya sendiri dan bertanggung jawab penuh atas seluruh siklus komunikasi dengan satu klien, mulai dari menerima permintaan hingga mengirimkan balasan dan akhirnya menutup sambungan.

Pendekatan ini menjamin skalabilitas dan responsivitas peladen yang tinggi.

---

## Pengujian dan Pertunjukan

Berikut adalah rekaman pelaksanaan program yang memperlihatkan kapabilitas peladen, baik untuk klien tunggal maupun klien serentak.

### 1. Uji Klien Tunggal (Dasar)

Pengujian awal dilakukan dengan satu klien untuk memverifikasi fungsionalitas dasar.

**Sesi Interaksi Klien (`mesin-2`):**
```bash
(base) jovyan@272794a38dc5:~/work$ telnet 172.16.16.101 45000
Trying 172.16.16.101...
Connected to 172.16.16.101.
Escape character is '^]'.
TIME
JAM 06:14:20
QUIT
Connection closed by foreign host.
```

**Catatan Aktivitas di Server (`mesin-1`):**
```log
2025-06-09 06:14:12,654 - [SERVER] - Server aktif dan mendengarkan di port 45000...
2025-06-09 06:14:16,771 - [SERVER] - Menerima koneksi baru dari ('172.16.16.102', 40036)
2025-06-09 06:14:16,771 - [SERVER] - Thread baru untuk klien ('172.16.16.102', 40036) dimulai.
2025-06-09 06:14:20,355 - [SERVER] - Menerima pesan dari ('172.16.16.102', 40036): 'TIME'
2025-06-09 06:14:20,356 - [SERVER] - Mengirim waktu '06:14:20' ke ('172.16.16.102', 40036)
2025-06-09 06:14:55,348 - [SERVER] - Menerima pesan dari ('172.16.16.102', 40036): 'QUIT'
2025-06-09 06:14:55,348 - [SERVER] - Klien ('172.16.16.102', 40036) meminta keluar. Koneksi ditutup.
2025-06-09 06:14:55,348 - [SERVER] - Thread untuk klien ('172.16.16.102', 40036) telah berhenti.
```

### 2. Uji Konkurensi (Dua Klien Serentak)

Guna membuktikan kemampuan *multithreading*, peladen diuji dengan dua klien yang terhubung pada saat yang sama dari `mesin-2` dan `mesin-3`.

**Sesi Interaksi Klien 1 (`mesin-2`):**
```bash
(base) jovyan@272794a38dc5:~/work$ telnet 172.16.16.101 45000
Trying 172.16.16.101...
Connected to 172.16.16.101.
Escape character is '^]'.
TIME
JAM 08:55:11
TIME
JAM 08:55:27
QUIT
Connection closed by foreign host.
```

**Sesi Interaksi Klien 2 (`mesin-3`):**
```bash
(base) jovyan@59f6059766dd:~/work$ telnet 172.16.16.101 45000
Trying 172.16.16.101...
Connected to 172.16.16.101.
Escape character is '^]'.
TIME
JAM 08:55:14
TIME
JAM 08:55:27
QUIT
Connection closed by foreign host.
```

**Catatan Aktivitas Serentak di Server (`mesin-1`):**
Catatan ini dengan jelas menunjukkan bagaimana peladen mengelola permintaan dari dua klien (`...102` dan `...103`) secara bergiliran tanpa saling menghalangi.
```log
2025-06-16 08:54:40,976 - [SERVER] - Server aktif dan mendengarkan di port 45000...
2025-06-16 08:55:03,200 - [SERVER] - Menerima koneksi baru dari ('172.16.16.103', 34758)
2025-06-16 08:55:03,201 - [SERVER] - Thread baru untuk klien ('172.16.16.103', 34758) dimulai.
2025-06-16 08:55:05,192 - [SERVER] - Menerima koneksi baru dari ('172.16.16.102', 58400)
2025-06-16 08:55:05,193 - [SERVER] - Thread baru untuk klien ('172.16.16.102', 58400) dimulai.
2025-06-16 08:55:11,352 - [SERVER] - Menerima pesan dari ('172.16.16.102', 58400): 'TIME'
2025-06-16 08:55:11,353 - [SERVER] - Mengirim waktu '08:55:11' ke ('172.16.16.102', 58400)
2025-06-16 08:55:14,235 - [SERVER] - Menerima pesan dari ('172.16.16.103', 34758): 'TIME'
2025-06-16 08:55:14,236 - [SERVER] - Mengirim waktu '08:55:14' ke ('172.16.16.103', 34758)
2025-06-16 08:55:27,117 - [SERVER] - Menerima pesan dari ('172.16.16.103', 34758): 'TIME'
2025-06-16 08:55:27,118 - [SERVER] - Mengirim waktu '08:55:27' ke ('172.16.16.103', 34758)
2025-06-16 08:55:27,869 - [SERVER] - Menerima pesan dari ('172.16.16.102', 58400): 'TIME'
2025-06-16 08:55:27,869 - [SERVER] - Mengirim waktu '08:55:27' ke ('172.16.16.102', 58400)
2025-06-16 08:55:32,255 - [SERVER] - Menerima pesan dari ('172.16.16.102', 58400): 'QUIT'
2025-06-16 08:55:32,256 - [SERVER] - Klien ('172.16.16.102', 58400) meminta keluar. Koneksi ditutup.
2025-06-16 08:55:32,256 - [SERVER] - Thread untuk klien ('172.16.16.102', 58400) telah berhenti.
2025-06-16 08:55:36,893 - [SERVER] - Menerima pesan dari ('172.16.16.103', 34758): 'QUIT'
2025-06-16 08:55:36,893 - [SERVER] - Klien ('172.16.16.103', 34758) meminta keluar. Koneksi ditutup.
2025-06-16 08:55:36,894 - [SERVER] - Thread untuk klien ('172.16.16.103', 34758) telah berhenti.
```

---

## Kode Sumber Lengkap (`server_time.py`)

Berikut adalah kode sumber akhir yang diterapkan dalam proyek ini.

```python
import socket
import threading
import logging
from datetime import datetime

# Konfigurasi logging untuk menampilkan informasi secara rapi dengan timestamp.
logging.basicConfig(level=logging.INFO, format='%(asctime)s - [SERVER] - %(message)s')

class ProcessTheClient(threading.Thread):
    """
    Kelas ini akan menangani setiap koneksi klien dalam thread-nya sendiri.
    Ini adalah kunci untuk membuat server dapat melayani banyak klien secara bersamaan.
    """
    def __init__(self, connection, address):
        self.connection = connection
        self.address = address
        threading.Thread.__init__(self)

    def run(self):
        logging.info(f"Thread baru untuk klien {self.address} dimulai.")
        try:
            while True:
                data = self.connection.recv(1024)
                if not data:
                    logging.warning(f"Klien {self.address} menutup koneksi tanpa perintah QUIT.")
                    break

                request_str = data.decode('utf-8').strip()
                logging.info(f"Menerima pesan dari {self.address}: '{request_str}'")

                if request_str == "TIME":
                    now = datetime.now()
                    current_time = now.strftime("%H:%M:%S")
                    response = f"JAM {current_time}\r\n"
                    self.connection.sendall(response.encode('utf-8'))
                    logging.info(f"Mengirim waktu '{current_time}' ke {self.address}")
                
                elif request_str == "QUIT":
                    logging.info(f"Klien {self.address} meminta keluar. Koneksi ditutup.")
                    break
                
                else:
                    error_msg = "PERINTAH TIDAK DIKENALI. Gunakan TIME atau QUIT.\r\n"
                    self.connection.sendall(error_msg.encode('utf-8'))
                    logging.warning(f"Perintah tidak valid dari {self.address}.")

        except ConnectionResetError:
            logging.error(f"Koneksi dengan {self.address} terputus secara paksa oleh klien.")
        finally:
            self.connection.close()
            logging.info(f"Thread untuk klien {self.address} telah berhenti.")


class Server(threading.Thread):
    """
    Kelas Server utama yang tugasnya hanya mendengarkan koneksi baru
    dan menyerahkannya ke thread ProcessTheClient.
    """
    def __init__(self, port):
        self.port = port
        self.my_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.my_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        threading.Thread.__init__(self)

    def run(self):
        self.my_socket.bind(('0.0.0.0', self.port))
        self.my_socket.listen(5)
        logging.info(f"Server aktif dan mendengarkan di port {self.port}...")

        while True:
            connection, client_address = self.my_socket.accept()
            logging.info(f"Menerima koneksi baru dari {client_address}")
            
            client_thread = ProcessTheClient(connection, client_address)
            client_thread.start()

def main():
    PORT = 45000
    server = Server(PORT)
    server.start()

if __name__ == "__main__":
    main()
```
