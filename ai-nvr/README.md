# AI NVR — Dahua + Frigate + Claude Haiku Hibrit Kamera Analiz Sistemi

100 IP kameralı bir Dahua NVR kurulumuna, **orijinal kayıt sistemini bozmadan**, alan yetki kontrolü, ilk-giriş tetikleyici ve tır/dorse renk kayıt yetenekleri ekleyen hafif AI katmanı.

## Durum

> **PoC aşaması.** Dokümantasyon yazılıyor, kod henüz scaffold edilmedi. Aşağıdaki yol haritasına bakın.

| Aşama | Durum |
|---|---|
| Mimari kararları | ✅ Tamam |
| Dokümantasyon | 🚧 Yazılıyor |
| Docker iskelet | ⬜ Bekliyor |
| Pilot (2–3 kamera) | ⬜ Bekliyor |
| Üretim (10 alan / 100 kamera) | ⬜ Bekliyor |
| Coral USB upgrade | ⬜ Türkiye tedarik bekliyor |

## Ne Yapar?

1. **Dahua NVR kayda devam eder** — bu sistem ona dokunmaz, sadece RTSP sub-stream'leri paralel okur.
2. **15 kamera Coral USB'de** Frigate ile lokal kişi/araç tespiti (10 oda + 5 kapı).
3. **Coral'a sığmayan kameralar Haiku ile desteklenir** (motion-triggered snapshot analizi). Donanım bütçesi maksimum $60'da sabit.
4. **Oda state machine'i**: alan boşken ilk giren kişiyi kaydeder, dolu alanda spam uyarısı vermez. İzleme süresi 1+ dakika.
5. **Kapı olayları**: her geçişte alarm, **saniye hassasiyetinde** giriş ve çıkış zamanı kaydedilir. E-posta ile **canlı izleme linki** gönderilir.
6. **Kamyon girişinde** Claude Haiku ile **çekici (tır) ve dorse rengini** ayrı kaydeder. Plaka okumaz.
7. **Olayları Dahua NVR'a alarm olarak** geri besler — orijinal DSS/SmartPSS panelinde görünür.

## Hedef Donanım

- Mevcut Linux Ubuntu 22.04 sunucu, 12 GB RAM (4 GB SnipeIT tarafından kullanılıyor)
- **PoC**: CPU-only Frigate (~8 GB RAM tampon)
- **Production**: + **1 adet** Coral USB Accelerator (~$60, Türkiye'den tedarik sonrası)
- **Donanım tavanı sabittir.** Daha fazla AI kamerası gerekirse maliyet sadece Haiku tarafında artar.

## 100 Kamera Tahsis Planı

| Grup | Kamera | Mekanizma | Aylık $ |
|---|---|---|---|
| **A**: Aktif izlenen alanlar (oda) | 10 | Frigate + Coral | (Coral'a dahil) |
| **B**: Kapılar (alarm + log + e-posta) | 5 | Frigate + Coral | (Coral'a dahil) |
| **C**: Düşük öncelik | 10 | Motion + Haiku snapshot | ~$11 |
| **D**: Sadece NVR kaydı | 75 | NVR kayıt, AI yok | $0 |

## Hedef Aylık Maliyet

| Senaryo | LLM çağrı/ay | Aylık |
|---|---|---|
| PoC (2–3 kamera) | ~300 | < $1 |
| Production (Grup A+B+C, 25 kamera) | ~14.500 | **~$18** |

Donanım yatırımı toplam: **$60** (1× Coral USB) + mevcut sunucu.

## Hızlı Başlangıç

> 🚧 Henüz aktif değil. `docs/03-setup.md` yazıldıktan sonra burası canlanır.

```bash
# Planlanan
cp .env.example .env
# .env içine: ANTHROPIC_API_KEY, DAHUA_*, RTSP URL'leri
docker compose up -d
```

## Dokümantasyon

| Dosya | İçerik |
|---|---|
| [`docs/01-architecture.md`](docs/01-architecture.md) | Sistem mimarisi, veri akışı |
| [`docs/02-hardware.md`](docs/02-hardware.md) | Donanım gereksinimleri, Coral yükseltme yolu |
| [`docs/03-setup.md`](docs/03-setup.md) | Adım adım kurulum |
| [`docs/04-zone-rules.md`](docs/04-zone-rules.md) | Alan kuralları, state machine, ilk-giriş tetikleyici |
| [`docs/05-dahua-integration.md`](docs/05-dahua-integration.md) | Dahua RTSP, HTTP alarm, ONVIF entegrasyonu |
| [`docs/06-llm-strategy.md`](docs/06-llm-strategy.md) | Claude Haiku promptları, tır/dorse renk analizi |
| [`docs/07-cost-analysis.md`](docs/07-cost-analysis.md) | Maliyet analizi, kıyaslamalar |
| [`docs/08-operations.md`](docs/08-operations.md) | İşletim, izleme, yedekleme, sorun giderme |
| [`docs/09-notifications.md`](docs/09-notifications.md) | E-posta bildirimleri + imzalı izleme linki + viewer servisi |
| [`ROADMAP.md`](ROADMAP.md) | PoC → Production milestone'ları |

## Mimari Özet

```
  ┌──────────────────────────────────────────────────────────────┐
  │                  Mevcut Sunucu (12 GB RAM)                   │
  │                                                              │
  │   [ Ubuntu 22.04 + SnipeIT ]  ────  ~4 GB                    │
  │                                                              │
  │   ┌──────────────  AI Stack  (~6 GB)  ─────────────────┐    │
  │   │                                                     │    │
  │   │   Frigate (CPU, sonra Coral USB) ───── 2.5 GB      │    │
  │   │   PostgreSQL (olay + renk log)  ───── 0.5 GB       │    │
  │   │   Bridge (zone state + LLM)     ───── 0.3 GB       │    │
  │   │   Mosquitto (MQTT)              ───── 0.05 GB      │    │
  │   │   Grafana (dashboard)           ───── 0.3 GB       │    │
  │   │                                                     │    │
  │   └─────────────────────────────────────────────────────┘    │
  └──────────────────────────────────────────────────────────────┘
              ▲                                ▲
              │ RTSP sub-stream                │ HTTP alarm
              │                                ▼
       10 Dahua IP kamera              Dahua NVR (orijinal kayıt)
       (100 kameradan 10'u izlenir)    (DSS/SmartPSS panel görür)
```

Detay için → [`docs/01-architecture.md`](docs/01-architecture.md).

## Lisans

Özel proje — iç kullanım. Lisans belirlenmedi.

## İletişim

Sahibi: İbrahim Sümbül (ibrahimsumbulll@gmail.com)
