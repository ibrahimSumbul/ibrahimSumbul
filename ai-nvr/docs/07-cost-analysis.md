# 07 — Maliyet Analizi

## Özet

Donanım bütçesi sabitlendi: **maksimum 1 adet Coral USB ($60)**. Coral kapasitesini aşan kameralar Haiku ile desteklenir.

| Kalem | PoC | Production (15 Coral + 10 Haiku) |
|---|---|---|
| **Donanım (tek seferlik)** | $0 (mevcut sunucu) | +$60 (1× Coral USB) |
| **Aylık LLM (Haiku)** | < $1 | ~$15–20 |
| **Aylık elektrik** | ~$1 | ~$3 (CPU + Coral) |
| **Aylık SMTP** | $0 (Gmail) | $0 |
| **Aylık toplam** | **~$1–2** | **~$18–25** |
| **Yıllık toplam (1. yıl)** | ~$24 | ~$240 + $60 = $300 |
| **Yıllık toplam (2. yıl)** | ~$24 | ~$240 |

> Kapsam genişlerse (kamera ekleyince) maliyet doğrusal Haiku artar — donanım yatırımı yok.

## Kıyaslama: Üç Yaklaşım

| Yaklaşım | İlk yatırım | Aylık | 1 yıl | 3 yıl |
|---|---|---|---|---|
| Saf bulut LLM (1 fps, 100 kamera) | $0 | $2.592.000 | $31 M | $93 M |
| Frigate + GPU (saf lokal) | $1.500 | ~$40 | $1.980 | $2.940 |
| **Bu proje: Hibrit (Coral + Haiku)** | **$60** | **~$18** | **~$300** | **~$540** |

> Bu proje en pahalı yaklaşımın **1/150.000**'i, en ucuz lokal alternatifin **1/5**'i.

## LLM Maliyeti Detay

### Per-çağrı

| Bileşen | Token | $/M | Maliyet |
|---|---|---|---|
| Sistem prompt (cached, %10 maliyet) | 400 | 0,80 | $0,000032 |
| Görüntü 640×480 | ~400 | 0,80 | $0,000320 |
| User msg | 50 | 0,80 | $0,000040 |
| Output JSON | 200 | 4,00 | $0,000800 |
| **Per çağrı toplam** | | | **~$0,0012** |

### Aylık (üretim senaryo — 15 Coral + 10 Haiku-only kamera)

**Grup A+B (15 kamera Coral'da)** — Haiku sadece zenginleştirme için:

| Olay tipi | Adet/gün | Çağrı/ay | Aylık $ |
|---|---|---|---|
| Tır+dorse renk | 20 | 600 | $0,72 |
| Anomali doğrulama | 10 | 300 | $0,36 |
| Yetkisiz alan (M8+) | 5 | 150 | $0,18 |
| Kapı geçişi enrichment (ops.) | 150 | 4.500 | $5,40 |
| **Alt toplam** | | **5.550** | **~$6,66** |

**Grup C (10 kamera Coral'a sığmadı, motion-triggered Haiku)** — her motion event Haiku'ya gider:

| Parametre | Değer |
|---|---|
| Motion event / kamera / gün | ~30 |
| Toplam motion events / ay | 10 × 30 × 30 = 9.000 |
| Haiku call başına | $0,0012 |
| **Alt toplam** | **~$10,80** |

| Test/debug | – | 100 | $0,12 |
| **Genel Toplam** | | **~14.650** | **~$17,60/ay** |

Güvenli pay: bütçe **$30/ay** olarak konur (.env), alarm $24'te.

### Grup C maliyet duyarlılığı

Motion event sayısı en büyük değişken. Karşılaştırma:

| Motion/kamera/gün | Aylık çağrı | Aylık $ |
|---|---|---|
| 15 (sakin alan) | 4.500 | $5,40 |
| 30 (orta) | 9.000 | $10,80 |
| 50 (yoğun) | 15.000 | $18,00 |
| 100 (problemli) | 30.000 | $36,00 |

Eğer Grup C kameraları çok motion üretiyorsa:
- Motion threshold'unu yükselt (küçük gürültüleri yoksay)
- Mesai dışı sadece aktif et
- O kamerayı Grup D'ye düşür (NVR-only)

## Donanım Maliyeti

### PoC (sıfır ek yatırım)

- Mevcut Ubuntu sunucu (12 GB RAM, SnipeIT ile paylaşımlı)
- Mevcut kameralar ve NVR
- Mevcut network
- **Toplam: $0**

### Production Upgrade

- **1× Coral USB Accelerator (kesin tavan)**: ~₺2.500–3.500 (≈ $60)
- Disk: mevcut yerde ~25 GB lazım, ayrı disk gerekmez
- (Opsiyonel) UPS: zaten sunucuda var varsayımıyla

> Ek Coral alınmaz. Daha fazla kamera AI'a sokulacaksa **maliyet Haiku tarafında doğrusal** artar; donanım maliyeti sabit kalır.

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
| Grup C kamerası "yağmurda titreşen yaprak" → her dk motion | Aylık +$10–40 | Motion threshold yükselt, gece-only mode |
| Kamera saatte 50 yanlış-pozitif "truck" | Aylık +$40 | Frigate `min_score` 0.7+, zone'lar dar |
| Anthropic fiyatları %50 arttı | Aylık $18 → $27 | Bütçe artır veya Grup C küçült |
| 15 alan → 30 alan büyüme | Aylık $18 → $35 | Hâlâ ucuz, donanım yok |
| API rate limit yendi | Hizmet kesintisi | Queue + retry + bütçe guard |
| Spam motion → SMTP rate hit | Mail kesintisi | Per-zone 30 mail/saat limit (bkz. `09-notifications.md`) |

## Karşılaştırma Tablosu (Ek)

Türk pazarındaki ticari alternatifler (yaklaşık, 2026):

| Çözüm | Yıllık maliyet | Esneklik |
|---|---|---|
| Hikvision DeepInView analitik | ₺50.000–120.000 lisans | Düşük |
| Avigilon ACC + AI | ₺200.000+ donanım+lisans | Orta |
| Bosch Intelligent Video Analytics | ₺150.000+ | Düşük |
| **Bu proje** | **~₺3.500 (tek seferlik) + ~₺2.000/yıl** | **Yüksek (açık)** |

10–30× ucuz, üstüne istenirse özelleştirilebilir.
