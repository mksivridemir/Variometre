# ESP32 Variometre - BMP280 (ESP-IDF Sürümü)

ESP32, BMP280 barometrik basınç sensörü ve Nokia 5110 (PCD8544) LCD ekran kullanılarak geliştirilmiş, düşük güç tüketimli, tam otonom dijital variometre projesi. Dikey hızı (vario) hesaplayarak yamaç paraşütü, yelkenkanat ve planör pilotları için hassas sesli (lift/sink) ve görsel bildirimler sağlar.

Bu proje tamamen **ESP-IDF v6.0.x** ortamında, FreeRTOS görev yapısı kullanılarak C++ dilinde yazılmıştır. 

## Öne Çıkan Özellikler

- 📏 **Otomatik QFE Kalibrasyonu**: Açılışta 10 adet ölçüm alarak ortam basıncını referans alır ve kalkış yüksekliğini 0.0m olarak sıfırlar.
- ↕️ **Hassas Variometre**: EMA filtresi ile gürültüden arındırılmış anlık dikey hız (m/s) hesaplaması.
- 🖥️ **Zengin Ekran Verisi**: 
  - Anlık Rakım (m) ve Dikey Hız (m/s)
  - Uçuş Süresi (Zamanlayıcı - SS:DD:SS)
  - Uçuş sırasındaki Maksimum (H) ve Minimum (L) Yükseklikler
  - Anlık hava sıcaklığı (°C)
- 🔘 **Çok İşlevli Tek Buton Kontrolü**:
  - **Uzun Basım (3 Saniye)**: Cihazı düşük güç tüketimli derin uyku (Deep Sleep) moduna geçirir veya uykudan uyandırır.
  - **Çift Tıklama**: Uçuş esnasında vario sesini tamamen kapatır veya açar.
- 🎵 **Gelişmiş Sesli Uyarılar (Buzzer)**: 
  - **Tırmanış (Lift)**: Vario hızıyla orantılı olarak bip seslerinin frekansı ve hızı artar (Bülbül efekti). (> +0.15 m/s)
  - **Çöküş (Sink)**: Vario hızıyla orantılı olarak kalınlaşan, park sensörü tarzı alçalma uyarıları. (< -0.15 m/s)
  - **Sessiz Bölge**: Düz uçuşta sessiz kalır (-0.15 m/s ile +0.15 m/s arası).
  - **Açılış Melodisi**: Açılışta 4 vuruşlu test melodisi.
- 🖼️ **Özel Açılış Logosu**: Cihaz başlatılırken "VARIOMETRE" temalı grafik logo gösterilir.
- ⚡ **Düşük Güç Tüketimi (Optimize Edilmiş)**:
  - Deep Sleep modunda ekran kapanır, gereksiz çevre birimleri kapatılarak pil tüketimi minimuma indirilir. Sadece butona basıldığında uyanır.
  - Sensör ve Ekran 5Hz (200ms) aralıklarla güncellenerek I2C/SPI trafiği azaltılmıştır.

## Donanım Bağlantı Şeması

### 1. BMP280 Basınç Sensörü (I2C)
| BMP280 Pin | ESP32 Pin |
|---|---|
| VCC | 3.3V |
| GND | GND |
| SDA | GPIO 21 |
| SCL | GPIO 22 |
| SDO | GND (I2C Adresi: 0x76 olarak ayarlanır) |

### 2. Nokia 5110 LCD (Yazılımsal SPI)
| Nokia 5110 Pin | ESP32 Pin | Açıklama |
|---|---|---|
| RST | GPIO 14 | Ekran Sıfırlama (Reset) |
| CE (CS) | GPIO 27 | Çip Seçimi (Chip Select) |
| DC | GPIO 26 | Veri/Komut (Data/Command) Seçimi |
| DIN (MOSI)| GPIO 25 | Seri Veri Girişi |
| CLK (SCLK)| GPIO 33 | Seri Saat (Clock) Girişi |
| BL (Light)| GPIO 32 | Arka Işık (Backlight) Kontrolü |
| VCC | 3.3V | Güç |
| GND | GND | Toprak |

### 3. Pasif Buzzer (Transistör Sürücülü)
| Sürücü Devre Pin | ESP32 Pin |
|---|---|
| Transistör BAZ (B) | GPIO 18 (LEDC PWM ile) - (Araya 1k-4.7k direnç koyun) |
| Transistör EMİTTER (E)| GND |
| Transistör KOLLEKTÖR (C)| Buzzer (-) Bacağına |
| Buzzer (+) Bacağı | 5V veya 3.3V (Güç Kaynağına) |

### 4. Kontrol Butonu
| Buton Pin | Bağlantı |
|---|---|
| Bacak 1 | GPIO 4 |
| Bacak 2 | 3.3V |
*(Not: GPIO 4 yazılımsal olarak Pull-Down ayarlıdır. Butona basıldığında GPIO 4 pini HIGH (3.3V) seviyesine çekilir.)*

## Kullanım Kılavuzu

1. **Cihazı Açma**: Cihaza güç verdiğinizde veya uyku modundayken **Butona 3 saniye basılı tuttuğunuzda** cihaz uyanır. Ekranda logo görünür ve açılış melodisi çalar.
2. **Otomatik Kalibrasyon**: Cihaz açılırken mevcut basıncı ölçerek referans alır ve "Sifirlaniyor" uyarısı verir. Uçuş öncesi kalkış noktasındayken açmanız tavsiye edilir.
3. **Uçuş Ekranı**: Ekranda uçuş süreniz, anlık rakım, tırmanış hızınız (m/s), maksimum/minimum yükseklikleriniz ve hava sıcaklığı anlık olarak takip edilebilir.
4. **Sesi Açma/Kapatma**: Uçuş esnasında vario sesini kapatmak veya tekrar açmak için butona **çift tıklamanız** yeterlidir.
5. **Cihazı Kapatma (Uyku)**: Kullanım bittiğinde cihazı kapatmak için butona **3 saniye basılı tutun**. Ekranda "Uyku Modu" yazar, cihaz kapanır ve derin uyku moduna geçerek pil tasarrufu sağlar.

## Yazılım Kurulumu ve Derleme

Proje standart ESP-IDF derleme araçlarını (CMake & Ninja) kullanır.

### Gereksinimler
- ESP-IDF v6.0.x kurulu ve ortam değişkenleri tanımlı olmalıdır.

### Derleme ve Yükleme 
Proje klasöründeyken terminal üzerinden (ESP-IDF prompt) şu adımları izleyin:

```powershell
# 1. Projeyi derleyin
idf.py build

# 2. ESP32'ye yükleyin (COM4 yerine kendi portunuzu yazın)
idf.py -p COM4 flash
```

## Proje Yapısı

```text
esp32-variometer/
├── CMakeLists.txt         # Ana CMake Yapılandırması
├── sdkconfig              # Proje ayarları
├── README.md              # Bu dosya
└── main/                  
    ├── CMakeLists.txt     
    ├── main.cpp           # Ana döngü, sensör yönetimi, uyku modu ve buton mantığı
    ├── config.h           # Pin konfigürasyonları, eşikler ve zamanlama ayarları
    └── nokia5110.h        # Optimize edilmiş LCD sürücüsü ve bitmap render fonksiyonları
```
