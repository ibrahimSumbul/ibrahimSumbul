# 01 — Mimari

## Tasarım İlkeleri

1. **Orijinal sistemi bozma.** Dahua NVR kayda devam eder, DSS/SmartPSS paneli aynen çalışır. AI katmanı sadece RTSP'yi okur ve geri alarm besler.
2. **Hibrit zekâ.** Frigate lokalde ucuz/hızlı detection yapar (kişi, araç, kamyon). Bulut LLM (Claude Haiku) **sadece** anlam katmanı gerektiren işlerde devreye girer (tır+dorse renk ayrımı, anomali doğrulama).
3. **Olay tetikli.** Sürekli analiz yok. Hareket → Frigate → state machine → gerekirse LLM.
4. **State machine + idempotent.** Aynı durum tekrar tekrar alarm üretmez. "Alan dolu" durumunda spam yok.
5. **Production-grade davran.** Tüm servisler container, restart policy `unless-stopped`, health check, log rotation, backup planı.

## Bileşenler

```
┌────────────────────────────────────────────────────────────────────┐
│                         Dahua Kameralar                            │
│              100 IP kamera (10'u aktif AI izlemede)                │
└──────────┬──────────────────────────────────────────────┬──────────┘
           │ RTSP main-stream                              │ RTSP sub-stream
           ▼                                               ▼
   ┌──────────────────┐                       ┌──────────────────────┐
   │   Dahua NVR(s)   │                       │   Frigate (Docker)   │
   │  - Tam kayıt     │                       │  - YOLOv8n (CPU/TPU) │
   │  - DSS/SmartPSS  │                       │  - Zone detection    │
   │  - HDD storage   │◄──────HTTP alarm──────┤  - MQTT publisher    │
   └──────────────────┘                       └──────────┬───────────┘
                                                         │ events
                                                         ▼
                                              ┌─────────────────────┐
                                              │     Mosquitto       │
                                              │     (MQTT broker)   │
                                              └──────────┬──────────┘
                                                         │ subscribe
                                                         ▼
                                              ┌─────────────────────┐
                                              │   Bridge servisi    │
                                              │  (Python, async)    │
                                              │                     │
                                              │  ├─ Zone state FSM  │
                                              │  ├─ Haiku client    │
                                              │  ├─ Dahua HTTP API  │
                                              │  └─ Snapshot store  │
                                              └────┬──────────┬─────┘
                                                   │          │
                              ┌────────────────────┘          └─────────┐
                              ▼                                          ▼
                    ┌──────────────────┐                       ┌───────────────────┐
                    │   PostgreSQL     │                       │  Claude Haiku API │
                    │  - zone_events   │                       │  (Anthropic)      │
                    │  - truck_events  │                       │                   │
                    │  - llm_usage     │                       │  - tır/dorse renk │
                    └────────┬─────────┘                       │  - anomali check  │
                             │                                 └───────────────────┘
                             ▼
                    ┌──────────────────┐
                    │     Grafana      │
                    │   (dashboard)    │
                    └──────────────────┘
```

## Veri Akışı: Tipik Bir Olay

### Senaryo 1: Boş alana ilk giriş

1. Frigate `cam_01`'de hareket görür, YOLO `person` etiketi `zone_depo` içinde tespit eder.
2. Frigate MQTT'ye event publish eder: `frigate/events {"type":"new", "after":{"label":"person", "current_zones":["zone_depo"]}}`
3. Bridge servisi event'i alır. State machine `zone_depo`'nun durumuna bakar:
   - `occupied=False` ise → **first_entry** olayı. DB'ye yaz, Dahua'ya alarm, state'i `occupied=True` yap.
   - `occupied=True` ise → sessiz, sadece `last_seen` güncelle.
4. 60+ saniye kişi gözükmezse → bridge zone'u tekrar `occupied=False` yapar, çıkış event'i yazar.

### Senaryo 2: Kamyon girişi

1. Frigate `cam_giris`'te `truck` tespit eder (YOLO).
2. Bridge event'i alır, daha önce bu olay için renk analiz edilmemişse:
3. Frigate snapshot endpoint'inden frame indirir (`/api/<camera>/latest.jpg`).
4. Snapshot'ı Claude Haiku'ya gönderir (base64), tır+dorse renk JSON'u ister.
5. Sonucu `truck_events` tablosuna yazar, snapshot'ı diskte tutar, LLM maliyetini `llm_usage` tablosuna log'lar.
6. Dahua'ya "kamyon giriş" alarmı tetikler.

## Network Topolojisi

- Kameralar: ayrı VLAN (örn. `192.168.10.0/24`), sadece NVR ve AI sunucusu erişebilir.
- AI sunucu: SnipeIT ile aynı VLAN'da olabilir ama Frigate kamera VLAN'ına da erişmeli.
- Anthropic API: outbound HTTPS (443) gereklidir.
- Dahua HTTP API: AI sunucudan NVR'a erişim (genelde 80/443).

## RAM Bütçesi (12 GB sunucu)

| Servis | Tahmini RAM |
|---|---|
| Ubuntu 22.04 + sistem | ~1 GB |
| SnipeIT (Apache + MySQL + PHP) | ~3 GB |
| Frigate (CPU mode, 10 kamera) | ~2.5 GB |
| PostgreSQL | ~0.5 GB |
| Bridge Python servisi | ~0.3 GB |
| Mosquitto | ~0.05 GB |
| Grafana | ~0.3 GB |
| **Toplam** | **~7.7 GB** |
| **Tampon** | **~4.3 GB** |

## Erişim Modeli

| Servis | Port | Network | Auth |
|---|---|---|---|
| Frigate Web UI | 5000 | Iç (LAN) | reverse-proxy basic auth |
| Grafana | 3000 | Iç (LAN) | Username/password |
| Postgres | 5432 | Sadece Docker network | DB user/pass |
| MQTT | 1883 | Sadece Docker network | DB user/pass |
| Bridge | yok | Docker internal | - |

## Hata Toleransı

| Senaryo | Davranış |
|---|---|
| Kamera offline | Frigate yeniden bağlanır (ffmpeg retry), bridge log'lar |
| Anthropic API down | Bridge yerel kuyrukta tutar, max 1 saat retry, sonra dead-letter |
| Postgres down | Bridge MQTT'den event alır ama yazamaz → critical log, alarm |
| Dahua API down | Alarm gönderimi kuyruğa girer, max 100 retry, sonra atılır |
| Disk dolu | Frigate eski snapshot'ları sil, log alarm |

Detaylar → [`docs/08-operations.md`](08-operations.md)
