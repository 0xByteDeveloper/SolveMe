# FINDINGS 03 — Gerçek PVP client binary analizi (S4): Rich header + string teşhisi

Tarih: 2026-11-14 (checkpoint turu #6). Kaynak: rafalohaki/Metin2-Work `client/` klasörü (tam PVP client paketi).
Araç: `/home/z/my-project/pedetect/build/pedetect_cli --json` (kendi PeDetect MVP aracımız).
Ham JSON: `research/pedetect_mt2client_main.json`, `research/pedetect_mt2client_config.json`.
Örnekler yerelde: `scripts/research/bin_samples/`.

---

## 1. MAIN — ReleaseMetin2ProjectClientFiles.exe (5,515,264 byte)

| Alan | Değer | Yorum |
|---|---|---|
| Mimari | x86 | Klasik 32-bit metin2 client |
| Linker | **14.28** | **VS2019 (16.8/16.9) link.exe** |
| Timestamp | 1612227122 = **2021-02-02** | 2021'de yeniden bağlanmış |
| Rich header | prodId=0xC7 build=41118 (count 4) | MSVC derleyici damgası mevcut |
| PDB yolu | `D:\Metin2 Server Files\mainline\Kraizy Updated Edition\Client\Source\M2Output\ReleaseMetin2ProjectClientFiles.pdb` | **"Kraizy Updated Edition"** server files paketi; dev disk düzeni sızıyor |
| Hint | Python (conf 0.70) | **PYTHON27.DLL (66 import)** → Python 2.7 gömülü |
| Bağımlılıklar | GRANNY2.DLL(56), MSS32.DLL(54), SPEEDTREERT.DLL(50), DEVIL.DLL(14), D3D8.DLL | Klasik Ymir DLL seti |
| /GS | ON | Stack cookie aktif |
| RTTI | ON | Aşağıdaki string kanıtına bak |
| CRT | Static (/MT) | |
| Bölümler | .text .rdata .data .data1 .rsrc .reloc | **.textbss/.msvcjmc YOK → Release** |
| Entropy | 6.742 | Packer yok |

### String taraması (binary içinde)

- **"CSingleton" x46** — hepsi RTTI tip descriptor'larında: `.?AV?$CSingleton@VIBackground@@@@`, `.?AV?$CDynam...`
  → Template singleton'lar (CSingleton<T>) RTTI ile binary'de tip adı olarak yaşıyor. /GR açık.
- **"NEVER DECLARED" YOK, "DECLARED MORE THAN ONCE" YOK, "FREED AT RUNTIME" YOK**
  → Bu client **globalmetin/NoMercy (MessageBox) soyundan DEĞİL**; assert-only modernize soy
  (ikevin127/NakiuS/Tiberiu96 tarzı Singleton.h). MessageBox varyantını kullanan PVP'lerde
  bu üç string Release'te bile binary'de kalır — ayırt edici teşhis budur.
- `"SendSequence"` python string'i YOK; `"SetPacketSequenceMode"` VAR
  → sequence sistemi Python'a açılmamış, yalnız C++ içi call graph'ta (FINDINGS_02'deki `call SendSequence` ile uyumlu).

## 2. CONFIG — config.exe (258,048 byte)

| Alan | Değer | Yorum |
|---|---|---|
| Linker | **6.0** | **VS6 (1998) dönemi link.exe** |
| Timestamp | 1165251365 = 2006-12-04 | Ymir dönemi orijinal araç |
| Rich | DanS mevcut ama comp.id çözülemedi (eski format) | |
| /GS | OFF | Eski araç zinciri |
| RTTI | ON (ama CSingleton tip string'i yok) | config.exe client sınıflarını içermez |

## 3. Anlamı — nesiller karışık "server files" gerçeği

Aynı PVP paketi içinde **iki nesil araç zinciri** yan yana: yardımcı araçlar (config.exe) VS6'dan
beri hiç yeniden derlenmemiş; ana client ise 2021'de VS2019 ile yeniden bağlanmış (Kraizy edition).
Kullanıcının "metin2 PVP'ler VS 20xx ile derleniyor" gözlemi doğru; **hangi soyda olduklarını
string taraması belirler**:

| Binary'de | Soy |
|---|---|
| `NEVER DECLARED` + kardeş stringler VAR | globalmetin/NoMercy (MessageBox) soyu |
| Stringler YOK ama `.?AV?$CSingleton@...` RTTI descriptor'ları VAR | modernize assert-only soy |
| RTTI descriptor'ları da YOK + eski CRT (msvcrt71/80/90d) | klasik Ymir orijinal |

## 4. Ertelenen alt hedef

SendSequence'ın kendi gövdesi için EterLib TU'sunun `.cod`'u NakiuS repo'sunda yok (94 .cod yalnız
UserInterface/Distribute altında). Gövde analizi S6'da final binary üzerinden (proc içi SEQ kurulum
kalıbı) yapılacak; FINDINGS_02'deki teşhis kuralı bu amaçla yeterli.
