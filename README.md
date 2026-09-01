# ⚡ ESP32 Industrial Compost Aeration & Thermal Decomposition Profiler

[![Lisensi: MIT](https://img.shields.io/badge/Lisensi-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Platform: ESP32](https://img.shields.io/badge/Platform-ESP32%20|%20FreeRTOS-blue.svg)](#)
[![Framework: Arduino IDE](https://img.shields.io/badge/Framework-Arduino%20IDE%202.0%2B-teal.svg)](https://www.arduino.cc/)
[![Status: Firmware Produksi](https://img.shields.io/badge/Status-Firmware%20Produksi-brightgreen.svg)](#)
[![Developer: Muhammad Fikri](https://img.shields.io/badge/Developer-Muhammad%20Fikri-blue.svg)](#)

Compost facility automation system monitoring 4-probe PT100 temperature gradients (thermophilic stage 55-65°C), moisture levels, and driving forced-air blowers.

---

## 📊 Diagram Blok Arsitektur & Skema Alur Rangkaian

Visualisasi interaktif alur daya, akuisisi sinyal sensor, pemrosesan algoritma inti, dan aktuasi proteksi perangkat:

```mermaid
graph TD
    subgraph Environmental_Sensing ["🌱 Sensor Pertanian & Lingkungan"]
        SOIL["Sensor Kelembaban Tanah Kapasitif"] --> SENS_IN["Sinyal Analog A0/ADC"]
        CLIMATE["Sensor BME280 / SHT31 (Suhu/Kelembaban)"] -->|"I2C Bus"| MCU["🧠 ESP32 (Dual-Core Xtensa LX6)"]
        FLOW["Water Flow Sensor (Hall Pulsa)"] -->|"Interupsi Pulsa"| MCU
        SENS_IN --> MCU
    end

    subgraph Agro_Logic ["🧠 Psychrometric & Irigasi FSM"]
        MCU -->|"Algoritma Agrikultur"| VPD["Kalkulasi Vapor Pressure Deficit (VPD)"]
        VPD -->|"Closed-Loop Dosing"| FSM["FSM Otomasi Irigasi & Aerasi"]
        FSM -->|"Non-Volatile"| NVS["Penyimpanan Ambang Batas Setpoint"]
    end

    subgraph Actuators_Field ["💧 Aktuator Lapangan"]
        FSM -->|"Relay Control"| PUMP["Pompa Air Irigasi & Solenoid Valve"]
        FSM -->|"Dosing Motor"| DOSER["Pompa Peristaltik Nutrisi AB Mix"]
        MCU -->|"I2C Monitor"| LCD["Layar LCD 20x4 Status Tanaman"]
        MCU -->|"IoT Cloud"| AGRO_CLOUD["Wi-Fi Smart Agri Dashboard"]
    end

    style MCU fill:#1565c0,stroke:#0d47a1,stroke-width:2px,color:#fff
    style VPD fill:#2e7d32,stroke:#1b5e20,stroke-width:2px,color:#fff
    style PUMP fill:#00838f,stroke:#006064,stroke-width:2px,color:#fff
```

---

## 📦 Daftar Komponen & Bahan Lengkap (Bill of Materials - BOM)

Berikut rincian spesifikasi komponen fisik dan modul yang dibutuhkan untuk membangun proyek ini:

| No | Nama Komponen / Modul | Estimasi Jumlah | Fungsi & Spesifikasi Teknis |
|:---|:---|:---|:---|
| 1 | **ESP32 Dev Module (30/38-Pin)** | 1 Unit | Mikrokontroler Dual-Core dengan FreeRTOS, Wi-Fi 2.4GHz & BLE |
| 2 | **Regulator Step-Down LM2596 / PSU 5V 2A** | 1 Unit | Sumber daya listrik 5V DC teregulasi |
| 3 | **Sensor Lingkungan Presisi (BME280 / SHT31 / EC & pH Probe)** | 2-3 Unit | Pengukur kelembaban tanah, suhu, pH, dan nutrisi EC |
| 4 | **Katup Solenoid Irigasi 12V DC / Pompa Peristaltik Dosing** | 2-4 Unit | Aktuator penyiraman presisi dan injeksi nutrisi |
| 5 | **Modul Relay Optocoupler Multi-Channel 5V** | 1 Unit | Pengendali daya pompa air, kipas aerasi, dan solenoid |
| 6 | **Layar LCD 20x4 I2C / OLED Display** | 1 Unit | Monitoring status kelembaban tanah dan siklus irigasi |
| 7 | **Sensor Flow Meter Hall Effect YF-S201** | 1 Unit | Pengukur debit dan volume air irigasi kumulatif |

---

## 🧠 Arsitektur Sistem & Fitur Utama

- **FreeRTOS Multi-Core Priority Scheduling:** Memisahkan pemrosesan sinyal presisi tinggi dari task telemetri untuk mencegah *latency jitter*.
- **Digital Signal Processing (DSP) & Filtering:** Dilengkapi algoritma digital filtering terdedikasi untuk eliminasi derau sinyal analog.
- **Non-Volatile Storage (NVS Preferences Flash):** Parameter kalibrasi, *setpoint*, dan konfigurasi tersimpan secara persisten terhadap siklus pemadaman daya.
- **Hardware Failsafe & Emergency Interlock:** Perlindungan otomatis jika terjadi anomali tegangan, kelebihan beban arus, atau pemicuan tombol *Emergency Stop*.
- **Industrial Telemetry & Diagnostics:** Pelaporan status operasional secara real-time via Serial/JSON stream.

---

## 🔌 Skema Pinout & Koneksi Hardware

| Komponen / Sinyal | Pin (ESP32) | Deskripsi Fungsi |
|:---|:---|:---|
| **Sensor Analog Input** | `GPIO 36 (ADC1)` | Jalur pembacaan sensor utama berpresisi tinggi |
| **Emergency Stop (E-Stop)** | `GPIO 34` | Pemicu pengaman darurat hardware interrupt |
| **Actuator / Relay Utama** | `GPIO 26` | Pengendali beban daya tinggi / relay aktuator |
| **Acoustic Alarm Buzzer** | `GPIO 25` | Indikator peringatan audible saat terjadi anomali |
| **Status / Heartbeat LED** | `GPIO 2` | Indikator status aktivitas sistem real-time |

---

## 🛠️ Panduan Perakitan Hardware (Langkah Demi Langkah)

1. **Persiapan Catu Daya:** Hubungkan catu daya utama ke jalur daya mikrokontroler. Pasang kapasitor *decoupling* 100nF di dekat pin VCC untuk meredam ripple switching.
2. **Pemasangan Sensor & Modul:** Sambungkan jalur sinyal sensor ke pin mikrokontroler yang telah ditentukan. Gunakan resistor pull-up 4.7kΩ pada jalur SDA/SCL jika menggunakan modul I2C.
3. **Pemasangan Aktuator:** Hubungkan modul relay / gate driver MOSFET ke pin kontrol output. Pasang dioda *flyback* (1N4007) pada beban induktif untuk mengeliminasi lonjakan tegangan balik (*back-EMF*).
4. **Pemasangan Tombol Emergency Stop:** Sambungkan tombol darurat ke pin interupsi eksternal dengan konfigurasi *Active-LOW* menggunakan resistor *pull-up*.
5. **Verifikasi Koneksi:** Lakukan pengecekan jalur ground bersama (*Common Ground*) pada seluruh modul sebelum menyalakan daya.

---

## 🚀 Panduan Kompilasi & Upload (Arduino IDE)

1. Buka **Arduino IDE 2.0+**.
2. Masuk ke menu **Tools > Board**:
   * Pilih **`ESP32 Dev Module`**.
3. Pastikan dependensi pustaka terpasang via Library Manager:
   * `ArduinoJson`
   * `Wire` & `SPI`
   * `Preferences`
4. Buka berkas [`esp32-smart-compost-reactor.ino`](./esp32-smart-compost-reactor.ino).
5. Klik tombol **Verify** (✓) kemudian **Upload** (➔).
6. Buka **Serial Monitor** pada baudrate **`115200`** untuk melihat streaming telemetri dan status operasional.

---

## 📄 Lisensi
Didistribusikan di bawah lisensi open-source **MIT License**. Dibuat dengan ❤️ oleh **Muhammad Fikri Dev**.
