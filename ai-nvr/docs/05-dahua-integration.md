# 05 — Dahua Entegrasyonu

Bu sistem Dahua NVR/kameralarla **iki yönlü** iletişir:
1. **Okuma**: Kameralardan RTSP stream (Frigate çeker)
2. **Yazma**: NVR'a HTTP alarm gönderme (orijinal panelde olay göstermek için)

> NOT: Dahua API'leri model/firmware'e göre değişir. Bu doküman tipik DSS/NVR4xx/5xx serileri içindir. PoC'nin ilk haftası tamamen API uyum testleri için harcanmalıdır.

## Bağlantı Stratejisi: NVR vs Doğrudan Kamera

NVR şu an **%50 yükte çalışıyor**. Üzerine sub-stream pull eklemek riskli olabilir; önce hesaplayalım.

### NVR Yük Tahmini

| Etki kalemi | Yaklaşık |
|---|---|
| Her RTSP sub-stream pull (640×480 @ 5 fps, H.264) | ~1–3% NVR CPU + ~256 kbps |
| 15 paralel stream | **+15–45% NVR CPU** |
| Snapshot endpoint sorgusu (anlık) | ~5% CPU/sorgu |
| H.265 → H.264 transcode (gerekirse) | +2–5%/stream |

Mevcut %50 + yeni ~%30 = **~%80**. Tehlikeli sınırın eteğinde; kayıt güvenilirliği etkilenebilir.

> **Karar**: Direct kamera bağlantısı **birinci tercih**. NVR üzerinden bağlantı **yedek**. NVR CPU %70'i geçerse derhal direct'e geç.

### Tercih Sıralaması

```
                  ┌───────────────────────────────┐
                  │  Kamera erişilebilir mi?      │
                  │  (statik IP, network açık?)   │
                  └───────┬─────────────┬─────────┘
                          │ Evet        │ Hayır
                          ▼             ▼
              ┌──────────────────┐  ┌──────────────────┐
              │  DIRECT bağlantı │  │  NVR üzerinden   │
              │  Kamera IP:554   │  │  channel=N       │
              │  NVR'a yük: 0    │  │  NVR yük: artar  │
              └──────────────────┘  └──────────────────┘
```

İdeal hibrit:
- Kritik 10 oda kamerası → direct
- 5 kapı kamerası → direct (yüksek olay frekansı)
- 10 Grup C kamerası → NVR üzerinden olabilir (motion sadece, az kullanır)

### NVR Yük İzleme (zorunlu)

`bridge/`'de NVR HTTP `/cgi-bin/magicBox.cgi?action=getCpuUsage` periyodik sorgulanır:

| NVR CPU | Aksiyon |
|---|---|
| < %60 | Sessiz |
| 60–70 | Log uyarı |
| 70–80 | Grafana alarm + Slack/email |
| > %80 | Otomatik stream sayısını azalt (Grup C devre dışı) |

`llm_usage` ile aynı tabloda `nvr_health` snapshot'ları tutulur.

## RTSP URL'leri

Standart Dahua RTSP path'i:

```
rtsp://<user>:<pass>@<ip>:554/cam/realmonitor?channel=<N>&subtype=<S>
```

| Parametre | Anlam |
|---|---|
| `channel=N` | Kamera kanalı (1-64). NVR'a doğrudan bağlanırken kanal. Kameraya doğrudan bağlanırken `channel=1`. |
| `subtype=0` | Main stream (yüksek çözünürlük, kayıt için) |
| `subtype=1` | Sub stream (düşük çözünürlük, AI detection için) |

### Doğrudan kameraya bağlanma

```
rtsp://admin:KAMERA_ŞİFRESİ@192.168.10.21:554/cam/realmonitor?channel=1&subtype=1
```

### NVR üzerinden bağlanma (channel ile)

```
rtsp://admin:NVR_ŞİFRESİ@192.168.10.10:554/cam/realmonitor?channel=3&subtype=1
```

> **NOT (channel adı)**: Dahua'da NVR kanalına isim verebilirsiniz ("DEPO_GIRIS"), ama RTSP URL'i hâlâ **rakam** ister. Channel adını URL'e koyamazsınız. Eşleştirmeyi config'de yapıyoruz:

```yaml
# bridge/config/cameras.yaml
nvr_channels:
  depo_giris:                   # bizim ad
    nvr_channel_id: 3           # Dahua kanal numarası
    nvr_channel_name: "DEPO_GIRIS"   # Dahua paneldeki ad (sadece referans)
    connection: nvr             # 'direct' veya 'nvr'
    ip: 10.0.0.10               # NVR IP veya kamera IP
  ana_kapi:
    nvr_channel_id: 12
    nvr_channel_name: "ANA_KAPI"
    connection: direct
    ip: 192.168.10.42
```

Bridge bu yaml'i okuyup `frigate/config.yml`'i otomatik üretebilir (M1'de).

### Sub-stream ayarı

Web UI → Camera → Stream → Sub Stream:
- Resolution: 640×480 (veya 704×576)
- FPS: 5–10
- Codec: H.264 (H.265 Frigate'te yavaş olabilir)
- Bitrate: 256–512 kbps yeterli

## NVR Orijinal Panelinde Ne Görünür?

Bizim AI sistemimizden gelen olaylar Dahua DSS/SmartPSS panelinde **"External Alarm"** olay tipi olarak görünür. Ayrıntı:

| Panel öğesi | Görünüm | Not |
|---|---|---|
| Live event listesi | `External Alarm — depo_giris — 14:32:18` | Anlık |
| Olay log (history) | Filtre: type=External Alarm | 30 gün retention default |
| Mobile push (DMSS app) | Title + snapshot | DMSS açık olmalı |
| Kayıt timeline marker | İşaretli (kırmızı çubuk) | Olay anına atlamak için tıklanabilir |
| Snapshot attach | Bazı NVR modelleri destekler | DSS Pro'da REST API ile zengin metadata |
| Açıklama metni | Bizim gönderdiğimiz string | Örn. "Boş depo'ya ilk giriş — kişi" |

### Dahua'nın Kendi AI Toolları + Bizim Sistem

Dahua NVR'ın **kendi AI motoru proprietary**'dir. Onun yerine geçmiyoruz, **tetikliyoruz**.

| Dahua AI Tool | Bizimle entegrasyon |
|---|---|
| IVS (perimeter, tripwire, intrusion) | Tetiklenebilir — bizim olayımız IVS event'i gibi gözükür |
| Face detection | Sadece Dahua kameralarında gömülü, dışarıdan extend edilemez |
| Vehicle / ANPR | Tetiklenebilir, snapshot eklenebilir (model bağlı) |
| Heat map | Bizim için sadece okunur |
| Custom event (**DSS Pro REST API**) | **En zengin entegrasyon** — başlık + açıklama + snapshot + tıklanabilir link |

### Tavsiye: Önce DSS Pro var mı sor

- DSS Pro varsa → Custom Event API kullan (en iyi UX)
- DSS Express / SmartPSS Plus → External Alarm + Virtual Input
- Saf NVR (DSS yok) → ONVIF Send Event veya Virtual Input

İlk hafta keşif: hangi NVR firmware'i + hangi DSS sürümü?

---

## HTTP Alarm Gönderme

Dahua "External Alarm" mekanizması, AI sisteminin olayları orijinal panelde göstermesini sağlar.

### Yöntem 1: HTTP CGI (NVR4xx/5xx)

```bash
curl -u admin:PASSWORD --digest \
  "http://<NVR_IP>/cgi-bin/configManager.cgi?action=setConfig&\
AlarmServer.0.Enable=true&\
AlarmServer.0.Address=<bridge_ip>&\
AlarmServer.0.Port=8080"
```

Bu **NVR'dan dışarıya** alarm gönderir. **Bizim ihtiyacımız tersi:** bridge'den NVR'a alarm.

### Yöntem 2: ONVIF Event Send (önerilen)

ONVIF üzerinden NVR'a "external trigger" eventi gönderebiliriz.

```python
# Pseudocode
from onvif import ONVIFCamera

nvr = ONVIFCamera("192.168.10.10", 80, "admin", "PWD")
event_service = nvr.create_events_service()
event_service.SendAuxiliaryCommand({"AuxiliaryCommand": "tt:Trigger/Alarm/0"})
```

### Yöntem 3: Virtual Alarm Input

Dahua bazı modellerde "Virtual Input" özelliği sunar. HTTP isteğiyle bir sanal input tetiklenir:

```
http://<NVR>/cgi-bin/alarm.cgi?action=triggerAlarm&channel=1&alarmType=External
```

NVR'da Setup → Event → Alarm → Local Alarm → Channel 1 → enable.

### Yöntem 4: SmartPSS Custom Event (en güvenli)

DSS Pro / SmartPSS Plus'ta REST API var:

```
POST https://<DSS>/admin/API/v1.0/customEvent
{
  "deviceCode": "CAM01",
  "eventType": "AI_DETECTION",
  "description": "Boş alana ilk giriş — depo_1",
  "snapshotUrl": "http://..."
}
```

### PoC için Önerilen Strateji

1. **İlk hafta**: Hangi yöntemin NVR'da çalıştığını test et. Yöntem 3 (Virtual Input) en yaygın çalışan.
2. **Üretim**: Yöntem 4 (SmartPSS Plus REST) varsa onu kullan, çünkü zengin metadata + snapshot URL desteği var.
3. **Fallback**: ONVIF (Yöntem 2)

## Authentication

Dahua çoğu API çağrısında **Digest Authentication** ister. Basic auth çalışmayabilir.

```python
import httpx
async with httpx.AsyncClient(auth=httpx.DigestAuth(user, pwd)) as client:
    r = await client.get(f"http://{nvr_ip}/cgi-bin/...")
```

## Bridge servisinde `dahua.py` arayüzü

```python
# Planlanan API
class DahuaClient:
    async def trigger_external_alarm(
        self,
        camera_channel: int,
        event_type: str,
        description: str,
        snapshot_path: str | None = None,
    ) -> None:
        """NVR'a external alarm event'i gönderir. Idempotent değil, her çağrıda tetikler."""
        ...

    async def get_capabilities(self) -> dict:
        """NVR'ın hangi event API'sini desteklediğini öğrenir, fallback için."""
        ...

    async def health_check(self) -> bool:
        ...
```

## Retry ve Queue

Dahua bazen 5xx döndürür veya zaman aşımına girer. Bridge:

- Max 3 deneme, exponential backoff (2s, 4s, 8s)
- 3'ü de başarısızsa: olay **DB'ye `alarm_pending=true`** olarak kaydedilir
- Arka plan worker pending alarmları 5 dakikada bir tekrar gönderir
- 1 saatten eski pending → dead-letter table, manuel inceleme

## Test Komutu (Milestone 4'te)

```bash
docker exec ainvr-bridge python -m bridge.dahua test-alarm \
  --camera cam_01 \
  --message "Test alarm from bridge"
```

Beklenen: 5 saniye içinde NVR'ın alarm logunda görülür + (varsa) mobile push.

## RTSP Sorunları için Hızlı Tanı

| Sorun | Çözüm |
|---|---|
| Bağlantı yok | `ffmpeg -i <rtsp_url> -t 5 -c copy test.mp4` ile elle test |
| H.265 yavaş | Kamerayı H.264'e geçir |
| Yüksek bandwidth | Sub-stream FPS'i düşür, çözünürlüğü azalt |
| Gecikme | `-rtsp_transport tcp` flag'i, UDP yerine TCP kullan |
| Ara sıra donma | `ffmpeg` retry config, Frigate `input_args` |

Frigate `input_args` örneği:

```yaml
ffmpeg:
  input_args: |
    -avoid_negative_ts make_zero
    -fflags +genpts+discardcorrupt
    -rtsp_transport tcp
    -timeout 5000000
    -use_wallclock_as_timestamps 1
```
