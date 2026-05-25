# Yol Haritası

Bu proje **PoC** olarak başlar, ama her milestone **production-ready** kalite hedefler. "Sonra düzeltirim" mantığı yok — her adım kalıcı olacak şekilde yazılır.

## Milestone 0: Dokümantasyon ve Mimari (mevcut)

- [x] Mimari kararları (Frigate + Haiku + Dahua bridge)
- [x] Donanım planı (PoC: CPU, prod: Coral USB)
- [x] Maliyet analizi
- [ ] Tüm `docs/` dosyaları tamam (8 dosya)
- [ ] Repository açılışı + draft PR

**Çıktı**: Bu repodaki `ai-nvr/` klasörü.

---

## Milestone 1: Local Stack İskeleti

**Hedef**: Tek komutla ayağa kalkan, henüz kameraya bağlı olmayan stack.

- [ ] `docker-compose.yml` — Frigate (CPU detector), Postgres, Mosquitto, Grafana, Bridge
- [ ] `.env.example` — tüm değişkenler
- [ ] `frigate/config.yml` — boş şablon, comment'lerle açıklamalı
- [ ] `bridge/` Python servis iskeleti (MQTT bağlanır, log atar)
- [ ] `db/schema.sql` — tablolar oluşur
- [ ] Health check: tüm container'lar `healthy`
- [ ] `make up` / `make down` / `make logs` Makefile

**Doğrulama**: `docker compose up -d && docker compose ps` → hepsi healthy.

---

## Milestone 2: Tek Kamera Pilot

**Hedef**: 1 Dahua kamera, 1 alan, ilk-giriş kuralı çalışıyor.

- [ ] Dahua kamerasından RTSP sub-stream alımı (Frigate config)
- [ ] Frigate person detection (CPU YOLOv8n)
- [ ] Bridge: MQTT person event dinler, log atar
- [ ] Zone state machine — TEK alan için boş→dolu geçişi
- [ ] PostgreSQL'e `zone_events` insert
- [ ] Manuel test: insan girince log, sürekli durunca tek event

**Doğrulama**: 10 dk içinde alana 3 kez girip çıkın → DB'de tam 3 `first_entry` kaydı olmalı.

---

## Milestone 3: LLM Entegrasyonu (Haiku)

**Hedef**: Tır rengi ve şüpheli olay doğrulama Haiku'ya gidiyor.

- [ ] Anthropic SDK + prompt caching kurulumu
- [ ] `bridge/llm.py` — tır renk analiz fonksiyonu
- [ ] JSON schema doğrulama (Pydantic)
- [ ] Retry + timeout + cost log (her çağrının maliyetini DB'ye yaz)
- [ ] Truck event flow: Frigate "truck" → snapshot al → Haiku → DB
- [ ] Rate limit guard (saatlik max çağrı)

**Doğrulama**: 1 kamyon görüntüsü ile manuel test, renk JSON çıkışı doğrulanır. Maliyet logu kontrol.

---

## Milestone 4: Dahua Alarm Köprüsü

**Hedef**: Olaylar orijinal Dahua panelinde alarm olarak görünüyor.

- [ ] Dahua HTTP alarm API araştırma + auth
- [ ] `bridge/dahua.py` — alarm gönderme fonksiyonu
- [ ] Zone first_entry → Dahua external alarm trigger
- [ ] Test: bridge'den alarm tetikle, Dahua DSS'te göründüğünü doğrula
- [ ] Failure handling: Dahua erişilemezse retry queue

**Doğrulama**: Bridge'den alarm tetiklenince Dahua mobile push gelir.

---

## Milestone 5: 10 Alan + Çoklu Kamera

**Hedef**: Production kapsamı, 10 izlenen alan, hepsinde state machine.

- [ ] Frigate config: 10 kamera tanımı
- [ ] Per-zone konfigürasyon (her alanın kendi kuralı)
- [ ] Performans test: CPU yükü, RAM kullanımı, gecikme
- [ ] **Coral USB değerlendirme** — CPU yetmiyorsa hemen sipariş tetiklenir
- [ ] Grafana dashboard: alan başına olay sayısı, LLM çağrı/maliyet

**Doğrulama**: 24 saat sürekli koşum, kaçırılan olay <%5, RAM stabil.

---

## Milestone 6: Coral USB Upgrade

**Hedef**: Türkiye'den Coral USB geldikten sonra TPU'ya geçiş.

- [ ] Coral driver kurulumu (libedgetpu)
- [ ] Frigate detector config değişikliği
- [ ] Performans karşılaştırma (önce/sonra CPU yükü)
- [ ] Dokümantasyon update

**Doğrulama**: Aynı yükte CPU kullanımı %30+ düşmeli.

---

## Milestone 7: Operasyonel Olgunluk

**Hedef**: Sistem unutulabilir hale gelsin.

- [ ] Backup stratejisi (Postgres + snapshot diski)
- [ ] Alert kuralları (Grafana → email/Telegram)
- [ ] Disk doluluk + LLM bütçe alarmı
- [ ] Log rotation
- [ ] Sistem restart senaryosu test
- [ ] Operasyon runbook (`docs/08-operations.md`)

**Doğrulama**: 1 hafta dokunmadan stabil.

---

## Milestone 8: Genişletme Fırsatları (opsiyonel)

- [ ] Yüz tanıma (CompreFace) — yetki kontrolü için
- [ ] Davranış/anomali tespiti (kavga, düşme)
- [ ] SnipeIT entegrasyonu (asset link)
- [ ] Mobile push notification
- [ ] Çoklu kullanıcı UI

---

## Risk & Karar Kayıtları

| Tarih | Karar | Gerekçe |
|---|---|---|
| 2026-05-25 | Frigate seçimi (saf bulut LLM yerine) | Aylık $2.5M maliyet yerine ~$5. Donanım bir kere. |
| 2026-05-25 | Coral USB ertelendi | Türkiye tedarik süresi var, PoC CPU ile başlayabilir. |
| 2026-05-25 | Plaka okuma kapsam dışı | Müşteri renk yeterli dedi. ALPR ayrı bir milestone. |
| 2026-05-25 | Yüz tanıma kapsam dışı | İlk fazda zone state machine yeter. M8'e bırakıldı. |
