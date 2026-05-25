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

**Tercih**: Coral USB. Sebep: PCIe slot şartı yok, takıp çalıştır.

### Coral USB'nin sağladığı

- Tek Coral: ~10–15 stream (low FPS) detection
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

Eğer kapsam büyürse:

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
