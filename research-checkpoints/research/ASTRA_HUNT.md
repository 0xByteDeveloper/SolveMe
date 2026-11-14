# ASTRA_HUNT — "astra" PVP kaynak/binary araması (S5)

Tarih: 2026-11-14 (cron turu). Amaç: kullanıcının bahsettiği "astra" PVP sunucusunun
kaynak kodunu veya client binary'sini bulmak (bkz. FINDINGS_03_binaries.md bağlamı).

## 1. GitHub repo aramaları (api.github.com/search/repositories, token'lı)

| Sorgu | Sonuç |
|---|---|
| `metin2 astra` | total=1 → **catalin1234121312/Metin2-Astralis** (stars=0, size=0KB, push=2025-09-24) |
| `astra2 metin2` | total=0 |
| `astra metin2 client` | total=0 |
| `astral metin2 source` | total=0 |
| `metin2 pvp astra` | total=0 |

`Metin2-Astralis` hâlâ **boş** (`GET /repos/.../contents/` → 404 "This repository is empty").
monitor.json'da izlemeye devam ediyor; source pushlanırsa zip otomatik alınacak.

## 2. GitHub code search (sleep 3 ile)

| Sorgu | Sonuç |
|---|---|
| `astra metin2 in:file` | total=4688 — tamamı gürültü (expired-domain listeleri, domain txt'leri) |
| `astra2 language:C++` | total=1472 — gürültü (Orbbec "Astra2" kamera SDK'sı baskın) |
| `SendSequence astra` | total=461 — gürültü (vscode/teleport/go terminal kodları) |

Tek metin2-temaslı isabet: `mt2-coder/mt2-account-aliases :: README.ro.md` — kontrol edildi,
Gameforge launcher hesap-alias eklentisi; "astra" kelimesi Rumence metinde tesadüfi geçiyor,
alakasız.

## 3. WebSearch — SOMUT LEAD: "Astra2" bir Türk PVP sunucusu

- **Turkmmo konu #3947753 — "[Unpack] Client Astra2 2023 & 2026 [PREMIUM]"**
  (forum.turkmmo.com/konu/3947753-unpack-client-astra2-2023-2026-premium, 2026-05-10):
  "sunucusuna ait **full unpack edilmiş Metin2 client dosyalarını** sunuyorum.
  Client, kaynak çıkarımı ve inceleme amacıyla paylaşılmıştır."
  → Bu, kullanıcının "astra" dediği sunucunun **client binary'sinin unpack halde mevcut** olduğunu gösteriyor.
- **Turkmmo — "Çalışıyor Astra2 Sınırsız Client Multifarm"** (2026-05-31):
  "Ekteki dosyayı **astra2 oyun klasörüne** koyun oyuna normal girin" → Astra2 aktif bir
  Türkçe Metin2 sunucusu; klasik client klasör yapısı (metin2client.exe + pack).
- Etiket sayfası: forum.turkmmo.com/tags/astra2-client (tek konu: yukarıdaki unpack).

## 4. Erişim denemesi — BAŞARISIZ (kısıtlar)

- WebFetch: sayfa Cloudflare challenge'ında; içerik görünmüyor.
- curl (browser UA): Cloudflare **error 1005 — sandbox IP'si site sahibi tarafından banlı**.
- İndirme ayrıca üyelik duvarı arkasında ("Öğeyi görmek için üye olmalısınız") + [PREMIUM] etiketi.
- Sonuç: bu turda Astra2 client binary'si indirilemedi.

## 5. Sonuç ve S6'ya bağlantı

- **GitHub'da Astra2/Astra kaynak kodu YOK** (repo + code search temiz negatif).
- **Turkmmo'da unpack edilmiş Astra2 client binary'si VAR** ama üyelik+Cloudflare duvarı
  nedeniyle sandbox'tan indirilemedi. Bu, elle indirilebilir → kullanıcı client exe'sini
  eliyle indirebilir; `pedetect_cli` Rich header analizi (FINDINGS_03 yöntemi) doğrudan
  uygulanabilir: Rich → derleyici sürümü → "NEVER DECLARED" soyu + /GL /LTCG teşhisi
  (S3b kuralı: Send* proc'unda son call'un varlığı/yokluğu).
- "astra" ≈ "Astra2" (Türk sunucu) ile "Metin2-Astralis" (boş repo) arasında isim benzerliği
  var; Astralis monitörde kalıyor.
- S6 FINAL_REPORT'ta bu lead "el ile edinilebilir kanıt" olarak listelenecek.
