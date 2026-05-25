# 05 — Dahua Entegrasyonu

Bu sistem Dahua NVR/kameralarla **iki yönlü** iletişir:
1. **Okuma**: Kameralardan RTSP stream (Frigate çeker)
2. **Yazma**: NVR'a HTTP alarm gönderme (orijinal panelde olay göstermek için)

> NOT: Dahua API'leri model/firmware'e göre değişir. Bu doküman tipik DSS/NVR4xx/5xx serileri içindir. PoC'nin ilk haftası tamamen API uyum testleri için harcanmalıdır.

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

### Doğrudan kameraya bağlanma (önerilen)

Her kameranın IP'si varsa doğrudan kameraya bağlanın. NVR'ı middleman yapmak gereksiz yük.

```
rtsp://admin:KAMERA_ŞİFRESİ@192.168.10.21:554/cam/realmonitor?channel=1&subtype=1
```

### NVR üzerinden bağlanma

Kamera IP'si yoksa NVR üzerinden çekin:

```
rtsp://admin:NVR_ŞİFRESİ@192.168.10.10:554/cam/realmonitor?channel=3&subtype=1
```

### Sub-stream ayarı

Web UI → Camera → Stream → Sub Stream:
- Resolution: 640×480 (veya 704×576)
- FPS: 5–10
- Codec: H.264 (H.265 Frigate'te yavaş olabilir)
- Bitrate: 256–512 kbps yeterli

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
