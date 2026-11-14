# FINDINGS 02 — nak_PhaseGame.cod tam taraması (S3b): call SendSequence imzası

Tarih: 2026-11-14 (checkpoint turu #4). Kaynak: `scripts/research/nak_PhaseGame.cod` —
NakiuS/Metin2Client Deposu'nun `PythonNetworkStreamPhaseGame.cpp` derlemesine ait MSVC `/FAcs` listing'i.
Derleyici: 18.00.40629.0 = MSVC 18.0 = **VS2013 Update 5** (FINDINGS_01b'de doğrulandı).
Tarama aracı: `scripts/cod_scan.py` → tam sonuç: `research/cod_scan_result.json`.

---

## 1. Sayısal sonuç

| Metrik | Değer |
|---|---|
| Toplam proc (Bu TU'da) | **240** |
| `?SendSequence@CNetworkStream@@QAE_NXZ` Bu TU'da tanımlı mı? | **HAYIR** (CNetworkStream = EterLib/NetStream TU'su) |
| Toplam `call SendSequence` | **53** |
| SendSequence çağıran proc | **53** (her birinde tam 1 call) |
| `Send*` ile başlayan proc | 56 |
| Tam 1 call SendSequence içeren Send* proc | **52** |
| Çağırmayan Send* proc (istisna) | **4** |

## 2. İstisna proc'lar (SendSequence ÇAĞIRMAYANLAR)

1. `?SendSpecial@CPythonNetworkStream@@QAE_NHPAX@Z` — COMDAT; düşük seviye gönderim yardımcısı (SendSequence muhtemelen kendi yolunu buraya bağımlı kılmıyor; döngü kırıcı istisna).
2. `?SendSyncPositionElementPacket@@QAE_NKKK@Z` — pozisyon senkron paketi; yüksek frekanslı pakette sequence baytı bilinçli eklenmiyor.
3. `?SendExchangeItemDelPacket@@QAE_NE@Z`
4. `?SendDragonSoulRefinePacket@@QAE_NEPAUSItemPos@@@Z`

Kod tarafı paraleli (FINDINGS_01): FALLEN1D kaynak taramasında da bazı Send* fonksiyonları `#ifdef` blokları içinde kalıyor/manuel true dönüyor — bu 4 istisna, kaynak varyasyonu ile uyumlu.

## 3. Kritik çıkarım — binary teşhis kuralı (S6 için hazır)

**Klasik (Gl'siz) derleme imzası:**
> Her `Send*Packet` proc'unun sonunda **tam bir adet `call ?SendSequence@CNetworkStream@@QAE_NXZ`** bulunur
> (istisna yukarıdaki 4 proc). SendSequence gövdesi hiçbir caller içinde GÖRÜNMEZ, çünkü:
> (a) proc Bu TU'da tanımlı değil → /Ob1'de TU içi inline da yok;
> (b) /GL yok → link-time cross-TU inline da yok.

**/GL + /LTCG derleme imzası (kullanıcının "karmaşa" fenomeni):**
> Bu `call` satırları **kaybolur**; SendSequence'ın gövdesi (SEQ paketi kurulumu) her Send* caller'ının
> disassembly'sine **cross-TU inline** olarak serpilir. `__forceinline Instance()` check'leriyle birlikte
> optimizer temel blokları yeniden sıralar → "SendAttackPacket'ın içinde SendSequence'ın içeriği,
> jne/je ile iç içe" görüntüsü. Teşhiste ilk bakılacak şey: proc sonu epilogdan hemen önce
> `call SendSequence` var mı / SEQ paketi kurulum kodu proc içine gömülmüş mü.

**Teşhis tablosu (binary → ayar):**

| Binary'deki tablo | Sonuç |
|---|---|
| Send* proc başına 1 adet `call SendSequence` | /GL'siz derleme (VS2013 tarzı, temiz) |
| call yok + SEQ kurulum kodu proc içinde + Instance check'leri serpiştirilmiş | /GL+/LTCG (NakiuS/Tiberiu vcxproj ayarları) |
| `SendSequence` simgesi hiç yok, import/public listesinde bile yok | Fully statically inlined (LTCG tam uygulanmış) |

## 4. Sınırlamalar ve sonraki adım

- `.cod` derleme anı listing'idir; /GL'de link-time inline'ı göstermez. Final binary davranışı için S4'te
  gerçek client exe'leri PeDetect + string/proc analizi ile doğrulanacak.
- SendSequence'ın kendi gövdesini görmek için EterLib TU'sunun .cod'u gerekir (NakiuS deposunda varsa
  S4 turunda çekilebilir — `EterLib/Distribute/NetStream.cod` benzeri yol).
- cod_scan.py yeniden kullanılabilir: başka .cod dosyalarına aynı analiz uygulanabilir.
