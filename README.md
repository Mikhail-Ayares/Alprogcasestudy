# Sensor Data Monitoring System - C++

Aplikasi ini merupakan sistem client-server untuk monitoring data sensor berbasis C++. Server menerima data dari client berupa JSON, melakukan deteksi anomali suhu, menyimpan data, dan menampilkan hasil pemrosesan secara periodik.

---

## 1. Struktur Proyek

```
├── anomalydetector.h
├── anomalydetector.cpp
├── common.h
├── datahandler.h
├── datahandler.cpp
├── client.cpp
├── server.cpp
```

---

## 2. Penjelasan Tiap Komponen

### `anomalydetector.h`

File header ini mendeklarasikan fungsi untuk deteksi suhu abnormal berdasarkan rentang tertentu. Di dalamnya didefinisikan nilai batas bawah dan atas suhu:

```cpp
const float minTemp = 20.0;
const float maxTemp = 26.0;
bool isAnomaly(float temperature);
```

### `anomalydetector.cpp`

File implementasi untuk `anomalydetector.h`. Fungsi `isAnomaly()` akan mengembalikan `true` jika suhu berada di luar batas wajar, dan `false` jika berada dalam rentang normal.

### `common.h`

Berisi struktur data umum yang digunakan di server dan client, seperti `SensorData` yang menyimpan timestamp, suhu, kelembapan, intensitas cahaya, dan ID pengirim.

```cpp
struct SensorData {
    std::string timestamp;
    float temperature;
    float humidity;
    int light;
    std::string client_id;
};
```

### `datahandler.h`

Header untuk kelas `DataHandler`, yang bertugas menyimpan semua data yang diterima dan menyediakan fungsi sorting dan pengambilan data:

* `addData(SensorData data)`
* `getSortedByTemperatureDescending()`
* `getSortedByTimestampAscending()`

### `datahandler.cpp`

Implementasi dari `DataHandler`. Data disimpan ke dalam `std::vector<SensorData>` lalu diurutkan menggunakan `std::sort` dengan lambda expression berdasarkan kebutuhan (temperatur atau timestamp).

### `client.cpp`

Program client yang:

1. Menghubungkan diri ke server (melalui socket)
2. Membuat data sensor dalam bentuk JSON
3. Mengirim data tersebut ke server secara periodik (misalnya setiap 5 detik)

### `server.cpp`

Program utama server. Tugasnya:

1. Menerima koneksi dari client
2. Membaca data JSON yang dikirim
3. Parsing menjadi objek `SensorData`
4. Mengecek apakah suhu data termasuk anomali menggunakan `isAnomaly()`
5. Menyimpan data ke dalam `DataHandler`
6. Menjalankan task periodik (misalnya tiap 10 detik) untuk:

   * Menampilkan Top 5 data berdasarkan suhu tertinggi
   * Menampilkan Top 5 data berdasarkan waktu terkini

---

## 3. Format Data JSON

Data dikirim client dalam format berikut:

```json
{
  "timestamp": "2025-06-13T21:46:31.903Z",
  "temperature": 31.94,
  "humidity": 50.42,
  "light": 542,
  "client_id": "CppClient-01"
}
```

---

## 4. Cara Kompilasi dan Menjalankan

### Kompilasi:

```bash
g++ server.cpp datahandler.cpp anomalydetector.cpp -o server -lws2_32
g++ client.cpp -o client -lws2_32
```

### Menjalankan:

* Jalankan server terlebih dahulu:

```bash
./server
```

* Kemudian jalankan client:

```bash
./client
```

---

## 5. Contoh Output Server

```text
[SERVER] Menerima data dari 127.0.0.1: T=31.94, H=50.4202, L=542, ID=CppClient-01
[PERINGATAN CEPAT] Peringatan (2025-06-13T21:46:31): Suhu (31.94 C) di luar batas normal [20.00, 26.00]

[SERVER TASK PERIODIK] Top 5 Data berdasarkan temperatur (DESC):
...
[SERVER TASK PERIODIK] Top 5 Data berdasarkan timestamp (ASC):
...
```

---

## 6. Catatan Tambahan

* Server dapat menerima data dari banyak client jika dikembangkan lebih lanjut.
* Program dapat dikembangkan lebih lanjut untuk menyimpan data ke file atau database.
* Tugas periodik dapat diatur interval waktunya dengan `std::this_thread::sleep_for()`.
