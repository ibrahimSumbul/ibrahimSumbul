# 07 — Maliyet Analizi

## Özet

| Kalem | PoC | Production (10 alan) |
|---|---|---|
| **Donanım (tek seferlik)** | $0 (mevcut sunucu) | +$60 (Coral USB) |
| **Aylık LLM (Haiku)** | < $1 | ~$2–5 |
| **Aylık elektrik** | ~$1 | ~$3 (CPU + Coral) |
| **Aylık toplam** | **~$1–2** | **~$5–8** |
| **Yıllık toplam (1. yıl)** | ~$24 | ~$120 + $60 = $180 |
| **Yıllık toplam (2. yıl)** | ~$24 | ~$120 |

## Kıyaslama: Üç Yaklaşım

| Yaklaşım | İlk yatırım | Aylık | 1 yıl | 3 yıl |
|---|---|---|---|---|
| Saf bulut LLM (1 fps, 100 kamera) | $0 | $2.592.000 | $31 M | $93 M |
| Frigate + GPU (saf lokal) | $1.500 | ~$40 | $1.980 | $2.940 |
| **Bu proje: Hibrit (10 alan + Haiku)** | **$60** | **~$5** | **~$120** | **~$240** |

> Bu proje en pahalı yaklaşımın **1/200.000**'i, en ucuz lokal alternatifin **1/10**'u.

## LLM Maliyeti Detay

### Per-çağrı

| Bileşen | Token | $/M | Maliyet |
|---|---|---|---|
| Sistem prompt (cached, %10 maliyet) | 400 | 0,80 | $0,000032 |
| Görüntü 640×480 | ~400 | 0,80 | $0,000320 |
| User msg | 50 | 0,80 | $0,000040 |
| Output JSON | 200 | 4,00 | $0,000800 |
| **Per çağrı toplam** | | | **~$0,0012** |

### Aylık (üretim senaryo)

| Olay tipi | Adet/gün | Çağrı/ay | Aylık $ |
|---|---|---|---|
| Tır+dorse renk | 20 | 600 | $0,72 |
| Anomali doğrulama | 10 | 300 | $0,36 |
| Yetkisiz alan (M8+) | 5 | 150 | $0,18 |
| Test/debug | – | 100 | $0,12 |
| **Toplam** | | **~1.150** | **~$1,40** |

Güvenli pay: bütçe **$20/ay** olarak konur (.env), alarm $16'da.

## Donanım Maliyeti

### PoC (sıfır ek yatırım)

- Mevcut Ubuntu sunucu (12 GB RAM, SnipeIT ile paylaşımlı)
- Mevcut kameralar ve NVR
- Mevcut network
- **Toplam: $0**

### Production Upgrade

- Coral USB Accelerator: ~₺2.500–3.500 (≈ $60–80)
- Disk: mevcut yerde ~25 GB lazım, ayrı disk gerekmez
- (Opsiyonel) UPS: zaten sunucuda var varsayımıyla

### Gelecek (12 ay sonra büyürse)

| Ekleme | Tahmini maliyet |
|---|---|
| Yüz tanıma (CompreFace + RTX 3060 12GB) | $300 |
| Davranış analizi (VideoMAE, RTX 4060 Ti) | $500 |
| 2. AI sunucu (yüksek kullanılabilirlik) | $800 |

## Elektrik Tüketimi

| Konfigürasyon | Güç (W) | Saat/ay | kWh/ay | TL/ay (₺3/kWh) | $/ay |
|---|---|---|---|---|---|
| Sunucu idle | 40 | 720 | 29 | ₺87 | ~$3 |
| + Frigate yükü (CPU) | 80 | 720 | 58 | ₺174 | ~$6 |
| + Coral USB | +5 | 720 | 62 | ₺186 | ~$6,2 |

Asıl yük zaten SnipeIT için ödeniyor, AI tarafının marjinal etkisi: ~$2–3/ay.

## ROI ve Karar

Sistem ne işe yarıyor (ekonomik değer)?

1. **İş güvenliği**: yetkisiz personel/araç girişi → potansiyel hırsızlık/sabotaj önleme
2. **Operasyonel görünürlük**: tır giriş çıkışı log → sevkıyat doğrulama
3. **Sorumluluk** (liability): alan ihlali kayıtları → sigorta/yasal koruma
4. **Yöneticiyi gece arayan telefonu azalt**: spam alarm yerine anlamlı alarm

**Geri ödeme**: Sistem 1 ciddi olayı (örn. 1 hırsızlık girişimi) yakalarsa kendini katlayarak öder.

## Riskler ve Bütçe Aşımı Senaryoları

| Senaryo | Maliyet etkisi | Önlem |
|---|---|---|
| Kamera saatte 50 yanlış-pozitif "truck" | Aylık $40 ekstra | Frigate `min_score` 0.7+, zone'lar dar |
| Anthropic fiyatları %50 arttı | Aylık $1.5 → $2.3 | Önemsiz, bütçe rahat |
| 10 alan → 30 alan büyüme | Aylık $5 → $15 | Hâlâ ucuz, GPU lazım olabilir |
| API rate limit yendi | Hizmet kesintisi | Queue + retry + bütçe guard |

## Karşılaştırma Tablosu (Ek)

Türk pazarındaki ticari alternatifler (yaklaşık, 2026):

| Çözüm | Yıllık maliyet | Esneklik |
|---|---|---|
| Hikvision DeepInView analitik | ₺50.000–120.000 lisans | Düşük |
| Avigilon ACC + AI | ₺200.000+ donanım+lisans | Orta |
| Bosch Intelligent Video Analytics | ₺150.000+ | Düşük |
| **Bu proje** | **~₺3.500 (tek seferlik) + ~₺2.000/yıl** | **Yüksek (açık)** |

10–30× ucuz, üstüne istenirse özelleştirilebilir.
