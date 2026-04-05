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

// =======================
// ISR (Interrupt Service Routine)
// =======================
void tombolDitekan() {
  unsigned long now = millis(); // Ambil waktu sekarang
  
  // Cek debounce (hindari trigger berulang dalam 200ms)
  if (now - lastInterruptTime > 200) {
    request = true;              // Set permintaan penyebrangan
    lastInterruptTime = now;     // Update waktu terakhir interrupt
  }
}

// =======================
// Fungsi Kuning Kedip 3x
// =======================
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

// =======================
// Setup (dijalankan sekali)
// =======================
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

// =======================
// Loop utama
// =======================
void loop() {

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
  
  // Jika tidak ada request, loop berhenti di sini
  if (!request) return;

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
}
