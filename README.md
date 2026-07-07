# ESP32 Variometre - BMP280 (ESP-IDF Sürümü)

ESP32, BMP280 barometrik basınç sensörü ve Nokia 5110 (PCD8544) LCD ekran kullanılarak geliştirilmiş, düşük güç tüketimli, tam otonom dijital variometre projesi. Dikey hızı (vario) hesaplayarak yamaç paraşütü, yelkenkanat ve planör pilotları için hassas sesli (lift/sink) ve görsel bildirimler sağlar.

Bu proje tamamen **ESP-IDF v6.0.x** ortamında, FreeRTOS görev yapısı kullanılarak C++ dilinde yazılmış olup, doğrudan kullanıma hazır şekilde derlenmiş (pre-compiled) olarak sunulmaktadır. 

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

## 🚀 Yazılım Kurulumu (Kod Derlemeye Gerek Yok)

Bu projeyi ESP32'nize yüklemek için herhangi bir kod derleme aracı veya programlama bilgisi gerekmez. Sağ taraftaki **Releases (Sürümler)** bölümünden en güncel `.zip` dosyasını indirerek doğrudan yükleme yapabilirsiniz.

İndirdiğiniz ZIP klasörünün içinde şu dosyalar bulunur:
- `bootloader.bin` (Yüklenecek Adres: **0x1000**)
- `partition-table.bin` (Yüklenecek Adres: **0x8000**)
- `esp32-variometer.bin` (Yüklenecek Adres: **0x10000**)

### Kurulum Yöntemi (Tarayıcı Üzerinden - En Kolay)

Cihazınıza hiçbir yazılım indirmeden Google Chrome veya Edge üzerinden doğrudan yükleyebilirsiniz:

1. [Adafruit ESPTool](https://adafruit.github.io/Adafruit_WebSerial_ESPTool/) web sitesine gidin.
2. ESP32'nizi USB ile bilgisayara bağlayın.
3. Sağ üstteki **"Connect"** butonuna tıklayıp ESP32'nizin bağlı olduğu portu seçin.
4. Alt kısımdaki dosya yükleme alanında **Offset** kısımlarına adresleri (`0x1000`, `0x8000`, `0x10000`) yazın.
5. Yanlarındaki **"Choose File"** butonlarına tıklayarak ZIP dosyasından çıkardığınız ilgili `.bin` dosyalarını seçin.
6. En alttaki **"Program"** butonuna basarak yüklemeyi tamamlayın.

Alternatif olarak Windows bilgisayarınızda **Espressif Flash Download Tool** masaüstü uygulamasını kullanarak da aynı adreslere bu dosyaları kolayca yazdırabilirsiniz.

İyi uçuşlar! 🪂
