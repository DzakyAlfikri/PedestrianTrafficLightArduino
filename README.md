# 🚦 Pedestrian Traffic Light Arduino UNO
```
Nama : Dzaky Alfikri
NIM : H1D023035
```
---

## Deskripsi Proyek
<img width="1148" height="643" alt="image" src="https://github.com/user-attachments/assets/eafbc445-1112-40fc-9dfb-bf9a07725ad1" />

https://www.tinkercad.com/things/idbjcm3w7cR-pedestrian-h1d023035?sharecode=R0szlMLizHKETDO0IW4masxhI1Bd0Cfi-wm14Zt9m3k


Proyek ini mensimulasikan sistem **traffic light penyeberangan pejalan kaki** menggunakan Arduino UNO. Sistem bekerja dengan prinsip lampu kendaraan default hijau, dan baru berpindah ke merah ketika pejalan kaki menekan tombol untuk menyeberang.

Terdapat **2 titik penyeberangan** (masing-masing dengan LED merah & hijau serta push button), sehingga pejalan kaki dari kedua sisi jalan dapat meminta untuk menyeberang.

---

##  Konsep Interrupt

**Interrupt** adalah mekanisme hardware pada microcontroller yang memungkinkan eksekusi kode terganggu saat event eksternal terjadi (dalam hal ini: tombol ditekan). Tanpa interrupt, program harus terus-menerus "polling" tombol untuk cek apakah ditekan atau tidak (tidak efisien). Dengan interrupt, Arduino dapat fokus pada tugas lain dan hanya bereaksi ketika event sebenarnya terjadi.

**ISR (Interrupt Service Routine)** adalah fungsi spesial yang dijalankan saat interrupt terjadi. Penting untuk menjaga ISR tetap singkat dan cepat agar tidak mengganggu operasi utama program. Di project ini, kedua tombol (pin 2 dan 3) terhubung ke hardware interrupt Arduino dengan mode FALLING (trigger ketika tombol ditekan dari HIGH ke LOW).

---

## Tujuan Project
1. **Keselamatan Pedestrian**: Memastikan pedestrian dapat menyebrang dengan aman ketika tombol diminta
2. **Pengelolaan Lalu Lintas**: Mengatur prioritas antara kendaraan dan penyebrangan pedestrian
3. **Implementasi Interrupt**: Mempelajari penggunaan hardware interrupt dan Interrupt Service Routine (ISR)
4. **Debounce Mechanism**: Mencegah false trigger pada tombol dengan mekanisme debounce
5. **State Management**: Mengelola status lampu lalu lintas berdasarkan kondisi sistem

---

### Komponen yang Digunakan

| Komponen | Jumlah | Keterangan |
|---|:---:|---|
| Arduino UNO | 1 | Mikrokontroler utama |
| LED Merah | 3 | 1 kendaraan + 2 pedestrian |
| LED Kuning | 1 | Transisi kendaraan |
| LED Hijau | 3 | 1 kendaraan + 2 pedestrian |
| Resistor 220Ω | 7 | Current limiter tiap LED |
| Push Button | 2 | Tombol penyeberangan |
| Breadboard | 1 | Media rangkaian |
| Kabel jumper | Secukupnya | Penghubung komponen |

### Konfigurasi Pin

| Pin Arduino | Fungsi |
|:---:|---|
| 2 | Push Button 1 (INPUT_PULLUP, Interrupt) |
| 3 | Push Button 2 (INPUT_PULLUP, Interrupt) |
| 4 | LED Merah Kendaraan |
| 5 | LED Kuning Kendaraan |
| 6 | LED Hijau Kendaraan |
| 7 | LED Merah Pedestrian 1 |
| 8 | LED Hijau Pedestrian 1 |
| 9 | LED Merah Pedestrian 2 |
| 10 | LED Hijau Pedestrian 2 |

---

## Cara Kerja

### Siklus Normal (Tanpa Request)
```
Kendaraan: HIJAU ON (jalan)
Pedestrian 1 & 2: MERAH ON (berhenti)
```

### Siklus Saat Tombol Ditekan
1. **Penghentian Kendaraan** 
   - Kendaraan: HIJAU → MERAH
   - Lampu merah kendaraan menyala

2. **Penyebrangan Pedestrian**
   - Pedestrian: MERAH → HIJAU
   - Pedestrian 1 & 2 dapat menyebrang selama 5 detik

3. **Transisi Kembali** 
   - Pedestrian: HIJAU → MERAH
   - Lampu kuning kendaraan berkedip 3 kali (transisi)
   - Total waktu kedip: 2 detik

4. **Pemulihan Normal** 
   - Kendaraan: MERAH → HIJAU (jalan lagi)
   - Sistem kembali ke kondisi awal

---

## Struktur Program

Program dirancang dengan pendekatan **modular dan event-driven** agar lebih rapi dan mudah dipahami:

### Komponen Utama

- **Deklarasi Pin Global** → Menyimpan mapping pin untuk kendaraan (merah, kuning, hijau) dan pedestrian (merah, hijau) dari kedua sisi, serta pin tombol (2 & 3)
- **Variabel State** → `request` (volatile bool) untuk flag permintaan penyebrangan, dan `lastInterruptTime` untuk mekanisme debounce
- **ISR `tombolDitekan()`** → Hardware interrupt handler yang dipanggil saat tombol ditekan (FALLING), dengan logika debounce 200ms
- **Fungsi Helper `blinkKuning3x()`** → Utility function yang membuat lampu kuning berkedip 3 kali sebagai sinyal transisi
- **Fungsi `setup()`** → Inisialisasi pin mode (OUTPUT untuk LED, INPUT_PULLUP untuk tombol) dan registrasi interrupt handlers
- **Fungsi `loop()`** → Main program yang menjalankan 3 fase secara siklis


###  Komponen Kode

| Komponen | Tipe | Fungsi |
|:---|:---:|---|
| `merah`, `kuning`, `hijau` | int (global) | Pin kendaraan traffic light |
| `ped1_merah`, `ped1_hijau`, `ped2_merah`, `ped2_hijau` | int (global) | Pin pedestrian lights |
| `button1`, `button2` | int (global) | Pin tombol (interrupt) |
| `request` | volatile bool | Flag request penyebrangan |
| `lastInterruptTime` | unsigned long | Timestamp debounce |
| `tombolDitekan()` | ISR | Interrupt handler dengan debounce |
| `blinkKuning3x()` | void function | Membuat kuning berkedip 3x |
| `setup()` | void function | Inisialisasi hardware |
| `loop()` | void function | Main program dengan 3 fase |

---

##  Penjelasan Kode

### 1. Deklarasi Pin & Variabel Global

```cpp
// =======================
// Traffic Light Kendaraan
// =======================
int merah  = 4;    // Pin 4 untuk lampu merah kendaraan
int kuning = 5;    // Pin 5 untuk lampu kuning kendaraan
int hijau  = 6;    // Pin 6 untuk lampu hijau kendaraan

// =======================
// Pedestrian 1
// =======================
int ped1_merah = 7;   // Pin 7 untuk lampu merah pedestrian 1
int ped1_hijau = 8;   // Pin 8 untuk lampu hijau pedestrian 1

// =======================
// Pedestrian 2
// =======================
int ped2_merah = 9;   // Pin 9 untuk lampu merah pedestrian 2
int ped2_hijau = 10;  // Pin 10 untuk lampu hijau pedestrian 2

// =======================
// Tombol Interrupt
// =======================
int button1 = 2;  // Pin 2 untuk tombol 1 (interrupt)
int button2 = 3;  // Pin 3 untuk tombol 2 (interrupt)

// Variabel global untuk request penyebrangan
volatile bool request = false;  
// 'volatile' digunakan karena variabel ini diubah di dalam ISR (interrupt)

// Variabel untuk debounce (mencegah tombol terbaca berkali-kali)
unsigned long lastInterruptTime = 0;
```

Bagian ini mendefinisikan semua pin yang dipakai. Pin 4–6 untuk traffic light kendaraan (merah, kuning, hijau), pin 7–8 untuk pedestrian 1, pin 9–10 untuk pedestrian 2, dan pin 2–3 untuk push button. Pin 2 dan 3 dipilih karena merupakan pin yang mendukung **hardware interrupt** pada Arduino UNO (`INT0` dan `INT1`). Variabel `request` bertipe `volatile bool`—keyword `volatile` **wajib** digunakan karena variabel ini diubah di dalam ISR (Interrupt Service Routine), sehingga compiler tidak akan mengoptimasi pembacaannya. Variabel `lastInterruptTime` digunakan untuk mekanisme **debounce** pada ISR.

---

### 2. Fungsi ISR `tombolDitekan()`

```cpp
void tombolDitekan() {
  unsigned long now = millis(); // Ambil waktu sekarang
  
  // Cek debounce (hindari trigger berulang dalam 200ms)
  if (now - lastInterruptTime > 200) { // debounce 200ms
    request = true;              // Set permintaan penyebrangan
    lastInterruptTime = now;     // Update waktu terakhir interrupt
  }
}
```

Fungsi ini adalah **Interrupt Service Routine (ISR)** yang dipanggil secara otomatis oleh hardware ketika salah satu tombol ditekan. Di dalamnya terdapat mekanisme **debounce** menggunakan `millis()`—jika jarak antara penekanan terakhir kurang dari 200ms, penekanan tersebut diabaikan. Hal ini mencegah pembacaan ganda akibat bouncing mekanis pada push button. Ketika valid, flag `request` di-set `true`.

---

### 3. Fungsi `blinkKuning3x()`

```cpp
void blinkKuning3x() {
  int jumlahKedip = 3;  // Jumlah kedipan
  int interval = 2000 / (jumlahKedip * 2); 
  // Total 2 detik dibagi ON dan OFF

  for (int i = 0; i < jumlahKedip; i++) {
    digitalWrite(kuning, HIGH); // Nyalakan lampu kuning
    delay(interval);            // Tunggu
    digitalWrite(kuning, LOW);  // Matikan lampu kuning
    delay(interval);            // Tunggu
  }
}
```

Fungsi khusus untuk membuat LED kuning **berkedip 3 kali** sebagai sinyal transisi. Interval kedipan dihitung dengan membagi total 2000ms dengan jumlah kedipan dikali 2 (karena tiap kedipan memiliki fase ON dan OFF). Dalam hal ini: `2000 / (3 * 2) = 333ms` per fase, jadi setiap siklus ON-OFF adalah 666ms. Loop `for` mengulangi siklus ON-OFF sebanyak 3 kali, memberikan visual warning kepada pengemudi sebelum lampu hijau menyala kembali.

---

### 4. Fungsi `setup()`

```cpp
void setup() {
  // Set pin lampu kendaraan sebagai OUTPUT
  pinMode(merah,  OUTPUT);
  pinMode(kuning, OUTPUT);
  pinMode(hijau,  OUTPUT);

  // Set pin pedestrian sebagai OUTPUT
  pinMode(ped1_merah, OUTPUT);
  pinMode(ped1_hijau, OUTPUT);
  pinMode(ped2_merah, OUTPUT);
  pinMode(ped2_hijau, OUTPUT);

  // Set tombol sebagai INPUT dengan pull-up internal
  pinMode(button1, INPUT_PULLUP);
  pinMode(button2, INPUT_PULLUP);

  // Pasang interrupt saat tombol ditekan (FALLING = dari HIGH ke LOW)
  attachInterrupt(digitalPinToInterrupt(button1), tombolDitekan, FALLING);
  attachInterrupt(digitalPinToInterrupt(button2), tombolDitekan, FALLING);
}
```

Inisialisasi mode pin. Semua LED diset sebagai `OUTPUT`, sedangkan kedua tombol diset sebagai `INPUT_PULLUP`, artinya menggunakan **resistor pull-up internal** Arduino sehingga tidak perlu resistor external. Selain itu, kedua tombol didaftarkan dengan `attachInterrupt()` menggunakan mode `FALLING`—artinya ISR `tombolDitekan()` akan dipanggil saat sinyal berubah dari `HIGH` ke `LOW` (saat tombol ditekan). Fungsi `digitalPinToInterrupt()` digunakan untuk mengkonversi nomor pin ke nomor interrupt yang sesuai.

---

### 5. Fungsi `loop()` - Loop Utama

Fungsi yang terus berjalan secara repetitif:

#### 5a. Kondisi Awal (Default State)

```cpp
// =======================
// Kondisi awal (default)
// =======================
digitalWrite(hijau,  HIGH);  // Kendaraan jalan (hijau)
digitalWrite(kuning, LOW);   // Kuning mati
digitalWrite(merah,  LOW);   // Merah mati

digitalWrite(ped1_merah, HIGH); // Pedestrian 1 berhenti
digitalWrite(ped1_hijau, LOW);

digitalWrite(ped2_merah, HIGH); // Pedestrian 2 berhenti
digitalWrite(ped2_hijau, LOW);
```

Setiap iterasi loop dimulai dengan menyalakan lampu **hijau kendaraan** dan **merah pedestrian** di kedua sisi. Ini adalah kondisi normal—kendaraan boleh lewat, pejalan kaki harus menunggu.

#### 5b. Pengecekan Flag Interrupt

```cpp
// Jika tidak ada request, loop berhenti di sini
if (!request) return;
```

Sistem memeriksa flag `request` yang di-set oleh ISR `tombolDitekan()`. Karena deteksi tombol ditangani oleh **hardware interrupt**, tidak perlu ada `digitalRead()` di dalam loop. Jika tidak ada request (tombol belum ditekan), `return` langsung ke awal loop—artinya sistem terus idle di kondisi default tanpa melakukan polling.

#### 5c. Fase Tombol Ditekan - Kendaraan Berhenti

```cpp
// =======================
// Saat tombol ditekan
// =======================

// Kendaraan langsung berhenti (merah)
digitalWrite(hijau, LOW);
digitalWrite(kuning, LOW);
digitalWrite(merah, HIGH);

// Pedestrian boleh menyebrang (hijau)
digitalWrite(ped1_merah, LOW);
digitalWrite(ped1_hijau, HIGH);

digitalWrite(ped2_merah, LOW);
digitalWrite(ped2_hijau, HIGH);

delay(5000); // Tunggu 5 detik
```

Ketika tombol ditekan (flag `request = true`), lampu kendaraan **langsung berubah ke merah**, dan kedua pedestrian light berubah menjadi **hijau**. Pejalan kaki diberi waktu **5 detik** untuk menyebrang.

#### 5d. Fase Transisi Kembali ke Normal

```cpp
// =======================
// Transisi kembali
// =======================

// Pedestrian kembali berhenti
digitalWrite(ped1_hijau, LOW);
digitalWrite(ped1_merah, HIGH);

digitalWrite(ped2_hijau, LOW);
digitalWrite(ped2_merah, HIGH);

// Kendaraan siap jalan lagi
digitalWrite(merah, LOW);

// Kuning berkedip sebagai tanda transisi
blinkKuning3x();

// Kendaraan jalan kembali
digitalWrite(hijau, HIGH);

// Reset request
request = false;
```

Setelah fase menyeberang selesai (5 detik), pedestrian kembali **merah**. Lampu kendaraan merah dimatikan, kemudian kuning berkedip 3 kali (total 2 detik) sebagai transisi sebelum lampu hijau menyala. Flag `request` di-reset ke `false`, sehingga loop kembali ke kondisi awal.

---

## Kesimpulan

Proyek ini mengimplementasikan sistem pedestrian traffic light yang bersifat **event-driven** menggunakan **hardware interrupt**—sistem hanya bereaksi ketika ada input dari pejalan kaki melalui push button. Penggunaan `attachInterrupt()` dengan mode `FALLING` memungkinkan deteksi tombol yang lebih responsif dibanding polling `digitalRead()`, karena interrupt langsung ditangani oleh hardware tanpa perlu menunggu loop selesai. Penggunaan `INPUT_PULLUP` menyederhanakan rangkaian karena tidak perlu resistor eksternal pada tombol, serta keyword `volatile` pada variabel `request` memastikan konsistensi data antara ISR dan loop utama.

---

