# 02 — Donanım

## Mevcut Durum (PoC için)

| Bileşen | Spec | Not |
|---|---|---|
| Sunucu | Linux Ubuntu 22.04 | SnipeIT halihazırda çalışıyor |
| RAM | 12 GB | 4 GB SnipeIT, 8 GB AI için müsait |
| CPU | (modeli teyit edilecek) | Frigate için en az 4 core x86_64 öneri |
| Disk | (boyutu teyit edilecek) | AI snapshot için ~50 GB yeterli |
| Network | LAN | Dahua kameralar + NVR ile aynı erişimde |
| GPU | Yok | PoC için gerekli değil |
| TPU | Yok (planlı) | Coral USB sonra eklenecek |

## PoC İçin Yeterli Mi?

**Evet, koşullu olarak.** Frigate CPU modunda 10 kamerayı **sub-stream'de düşük FPS** ile işleyebilir.

CPU detector ile beklenen yük (10 kamera, 640×480 @ 5fps, YOLOv8n):
- Tek detection: ~80–150 ms (modern x86 CPU)
- Toplam yük: ~%30–50 CPU sürekli
- Risk: tepe trafiği (örn. ardışık 5 hareket) gecikme yaratabilir

> Eğer CPU kullanımı %70'i aşıyorsa Coral USB'yi beklemeden FPS'i 3'e düşürün veya kamera sayısını azaltın. Doğru çözüm Coral USB.

## Coral USB Upgrade Yolu

| Ürün | Fiyat (US) | Türkiye yaklaşık | Stok |
|---|---|---|---|
| Coral USB Accelerator | $60 | ₺2.500–3.500 | Hepburn/Robotistan/Direnc |
| Coral M.2 (mini PCIe) | $40 | yok/zor | İthalat gerekir |

**Bütçe kararı**: **Maksimum 1 adet Coral USB ($60)**. Ek Coral alınmaz. Coral'a sığmayan kameralar Haiku ile desteklenir (aşağıdaki kapasite tablosu).

### Coral USB Kapasitesi (Gerçekçi)

Tek Coral USB, Edge TPU üzerinde **~100 inference/saniye** yapar. Kamera başına FPS düşürüldüğünde:

| FPS / kamera | Maks kamera | Notu |
|---|---|---|
| 10 fps | 8–10 | Hızlı reaksiyon |
| 5 fps | **15** | **Önerilen kapasite** |
| 3 fps | 20–25 | Reaksiyon gecikir |
| 2 fps | 30+ | Sadece olay tetikçi |

> **Karar**: 5 fps'te **15 kamera Coral üzerinde**, geri kalanlar motion-triggered Haiku ile çalışır.

### 100 Kamera için Tahsis Planı

| Grup | Kamera | Mekanizma | Maliyet |
|---|---|---|---|
| **A**: Aktif izlenen alanlar (oda) | 10 | Frigate + Coral (state machine) | TPU |
| **B**: Kapılar (alarm + giriş/çıkış log) | 5 | Frigate + Coral (door traversal) | TPU |
| **C**: Düşük öncelik (motion → Haiku) | 10 | ffmpeg motion + Haiku snapshot | LLM |
| **D**: Sadece NVR kaydı | 75 | NVR kayıt, AI yok | $0 |
| **Toplam** | **100** | | |

Grup A+B: **15 kamera Coral'da**, kapasiteye sığar.
Grup C: Coral kullanmaz, hareket halinde Haiku çağrısı yapar.
Grup D: Hiç AI işlemi yapılmaz, sadece Dahua NVR kaydı.

> Grup C'nin sayısı bütçeyle dengelenir. Bkz. [`07-cost-analysis.md`](07-cost-analysis.md).

### Coral USB'nin sağladığı

- Tek Coral: ~15 stream (5 fps) detection
- Inference süresi: ~7–10 ms (CPU'nun ~10 katı hızlı)
- CPU yükünü ~%70 azaltır
- Daha hızlı reaksiyon, daha az gecikme

### Kurulum (Coral geldiğinde)

```bash
# Ubuntu 22.04
echo "deb https://packages.cloud.google.com/apt coral-edgetpu-stable main" \
  | sudo tee /etc/apt/sources.list.d/coral-edgetpu.list
curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
sudo apt update
sudo apt install libedgetpu1-std

# USB takıldıktan sonra
lsusb | grep Google  # "Google Inc." görmek lazım
```

Sonra `frigate/config.yml` içinde:

```yaml
detectors:
  coral:
    type: edgetpu
    device: usb
```

## Üretim Donanım Hedefi (12 ay sonra)

**Bütçe sabitlendi**: ek donanım alınmaz. Kapsam genişlerse Haiku üzerinden ölçeklenir (maliyeti çok küçük artar).

İleride donanım yatırımı yapılacaksa:

| Senaryo | Donanım |
|---|---|
| 20+ aktif izleme alanı | + 2. Coral USB veya RTX 4060 8GB |
| Yüz tanıma eklenirse | + RTX 3060 12GB (CompreFace için) |
| Davranış analizi eklenirse | + RTX 4060 Ti 16GB |

## Disk Planı

| Veri | Tahmini boyut | Tutma süresi |
|---|---|---|
| Olay snapshot (JPG) | ~100 KB × 50 olay/gün | 90 gün → ~450 MB |
| Olay clip (mp4, opsiyonel) | ~5 MB × 50 olay/gün | 30 gün → ~7.5 GB |
| Postgres DB | ~50 MB/ay | 5 yıl → ~3 GB |
| Frigate cache | değişken | 7 gün → ~10 GB |
| **Toplam** | | **~25 GB** rahat |

50 GB ayırın, rahat olur. Mevcut diskte yer varsa ayrı disk gerekmez.

## Ağ Gereksinimleri

- Outbound 443 (Anthropic API)
- Kamera VLAN'ına Frigate erişimi
- NVR'a HTTP/HTTPS erişimi (alarm push)
- Internet bant genişliği: Haiku çağrıları küçük (her biri ~50 KB), 100 çağrı/saat = ~5 MB/saat. Sıkıntı yok.
