# 04 — Alan Kuralları ve State Machine

Bu sistemin en kritik mantığı: **gereksiz alarm üretme, ama gerçek olayı kaçırma.**

## Müşteri Kuralları (özet)

1. 100 kamera kayıt yapar ama **sadece 10 alan** AI ile izlenir.
2. Bazı alanlarda: alan içinde **biri varken uyarı verme.**
3. Alan **boşken, ilk gelen kişi** kayda alınır ve alarm üretir.
4. İzleme süresi **1 dakika veya üstüne** çıkabilir (kişi alanda kaldığı sürece).
5. Kişi ayrılıp tekrar girince → yeni "ilk giriş" sayılır.

## State Machine Tanımı

Her alan üç durumdan birinde olur:

```
                  ┌──────────────────────────────────┐
                  │                                  │
                  ▼                                  │
            ┌──────────┐                       ┌─────┴──────┐
            │  EMPTY   │   person detected     │  OCCUPIED  │
            │          │ ────────────────────► │            │
            │          │                       │            │
            └──────────┘                       └─────┬──────┘
                  ▲                                  │
                  │  no person for                   │
                  │  EXIT_TIMEOUT (60s)              │
                  └──────────────────────────────────┘
```

Ek olarak bir geçici durum: `EXIT_PENDING`

- Frigate bir kareye kadar kişi göstermeyince hemen `EMPTY` denmez.
- 60 sn boyunca kişi görülmezse → `EXIT_PENDING` → `EMPTY`.
- Bu, kişinin kameranın kör noktasına geçmesinden kaynaklı yanlış-pozitif çıkışları engeller.

## Olay (Event) Tipleri

| Olay | Tetikleyici | Kaydedilir mi? | Alarm üretir mi? |
|---|---|---|---|
| `first_entry` | EMPTY → OCCUPIED | ✅ Evet | ✅ Evet (Dahua) |
| `still_present` | OCCUPIED durumda 60 sn'de bir | ✅ Evet (heartbeat) | ❌ Hayır |
| `exit` | OCCUPIED → EMPTY | ✅ Evet | ❌ Hayır (opsiyonel) |
| `unauthorized` | OCCUPIED ama yetki yok* | ✅ Evet | ✅ Evet |

*Yetki kontrolü ileriki milestone'da, ilk fazda devre dışı.

## Konfigürasyon Şeması

`bridge/config/zones.yaml`:

```yaml
zones:
  - name: zone_depo_1
    camera: cam_01
    frigate_zone: zone_depo
    rules:
      enabled: true
      first_entry_alarm: true
      exit_alarm: false
      exit_timeout_seconds: 60
      min_person_score: 0.6
      # Bir nevi mesai saati filtresi
      active_hours: "00:00-23:59"   # 7/24
      # Bu alanda kimse yokken alarm üret
      alert_on_empty_arrival: true

  - name: zone_yukleme
    camera: cam_02
    frigate_zone: zone_loading
    rules:
      enabled: true
      first_entry_alarm: true
      exit_timeout_seconds: 120     # Yükleme alanı daha uzun
      min_person_score: 0.6
      active_hours: "18:00-08:00"   # Sadece mesai dışı alarm

  - name: zone_kamyon_giris
    camera: cam_giris
    frigate_zone: arac_giris
    rules:
      enabled: true
      track_objects: [truck, car]
      truck_color_analysis: true    # Haiku ile renk
      first_entry_alarm: true
```

## Bridge State Tutma

State **bellekte** tutulur (Python dict), ama her durum değişikliği DB'ye yazılır. Servis restart olunca son durum DB'den geri yüklenir.

```python
# Pseudocode
class ZoneStateMachine:
    def __init__(self, zone_config, db, llm, dahua):
        self.cfg = zone_config
        self.state = "EMPTY"
        self.since = utcnow()
        self.last_seen = None
        self.db = db
        self.llm = llm
        self.dahua = dahua

    async def on_person_event(self, event):
        if event.score < self.cfg.min_person_score:
            return

        now = utcnow()
        if self.state == "EMPTY":
            await self._handle_first_entry(event, now)
        else:
            self.last_seen = now  # heartbeat

    async def _handle_first_entry(self, event, now):
        self.state = "OCCUPIED"
        self.since = now
        self.last_seen = now

        snapshot = await fetch_snapshot(event.camera, event.frigate_event_id)
        await self.db.insert_zone_event(
            zone=self.cfg.name,
            event_type="first_entry",
            ts=now,
            snapshot_path=snapshot,
            metadata={"score": event.score, "frigate_id": event.frigate_event_id},
        )
        if self.cfg.first_entry_alarm and self._is_active_hour(now):
            await self.dahua.trigger_external_alarm(self.cfg.camera, "person_entered")

    async def tick(self):
        """Her 10 saniyede bir çağırılır."""
        if self.state != "OCCUPIED":
            return
        if (utcnow() - self.last_seen).total_seconds() > self.cfg.exit_timeout_seconds:
            self.state = "EMPTY"
            await self.db.insert_zone_event(
                zone=self.cfg.name,
                event_type="exit",
                ts=utcnow(),
            )
```

## Race Condition Tehlikeleri ve Çözüm

| Tehlike | Çözüm |
|---|---|
| Frigate aynı kişi için 100 event basar | `frigate_event_id` ile dedupe |
| Kişi 5 sn kameradan kayboldu, geri geldi | `exit_timeout` (60 sn) tampon olur |
| İki bridge instance | Tek instance kuralı, `Singleton` lock dosyası |
| Servis restart sırasında alan dolu | DB'den son state'i yükle |

## Yetki Kontrolü (M8'de eklenir)

İleride yüz tanıma eklendiğinde:

```yaml
zones:
  - name: zone_kasa
    rules:
      authorization_required: true
      authorized_persons: [muhasebe_1, muhasebe_2, mudur]
```

Yetkisiz kişi tespit edilirse `unauthorized` event ve özel alarm.

## Test Senaryoları

| Test | Beklenen sonuç |
|---|---|
| Boş alana 1 kişi girer | 1 `first_entry`, 1 Dahua alarm |
| 30 sn boyunca alanda durur | Sadece `still_present` heartbeat, ek alarm yok |
| Alan içinde 2. kişi girer | Ek alarm yok (zaten OCCUPIED) |
| Tüm kişiler çıkar | 60 sn sonra `exit` event |
| Çıkıştan 30 sn sonra geri gelir | Yeni `first_entry` (ama state hala EMPTY değildi → fix: 60 sn'lik exit_timeout dolmadan tekrar geldi → state hala OCCUPIED, alarm yok) |
| Çıkıştan 90 sn sonra geri gelir | Yeni `first_entry` ve yeni alarm |

Son satırdaki ince noktayı doğru anlamak için: **alarm sadece EMPTY → OCCUPIED geçişinde üretilir.**
