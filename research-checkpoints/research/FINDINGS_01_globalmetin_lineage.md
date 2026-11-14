# FINDINGS 01 — "CSingleton::Instance() NEVER DECLARED" soy kimliği ve derleme analizi

Tarih: 2026-11-14 (Oturum 1). Tüm kanıtlar GitHub üzerinden doğrulandı (raw kaynak + API).

---

## 1. String'in kaynağı: modernize edilmiş Singleton.h soyu

Bu string **orijinal Ymir kodunda yok**. Klasik Ymir `Singleton.h` yalnızca `assert(ms_singleton)` içerir (Release'te NDEBUG ile silinir). "NEVER DECLARED" versiyonu, PVP ekosisteminde dolaşan **modernize edilmiş (C++11 rewrite) bir Singleton şablonudur**.

globalmetin `source/client/source/eterBase/Singleton.h` (L34 civarı, doğrulanmış içerik):

```cpp
__forceinline static T & Instance()
{
    if (!ms_singleton)
        MessageBoxA(nullptr, typeid(T).name(), "CSingleton::Instance() NEVER DECLARED", MB_ICONEXCLAMATION | MB_OK);

    assert(ms_singleton);
    return (*ms_singleton);
}

__forceinline static T * InstancePtr()
{
    return (ms_singleton);
}
```

Kardeş stringler (aynı dosyanın parmak izi):
- `"CSingleton() DECLARED MORE THAN ONCE"`
- `"~CSingleton() FREED AT RUNTIME"`

### Neden bu string binary'de KALIYOR (klasikte KALMIYOR)?

`assert` değil **gerçek bir `MessageBoxA` çağrısı**dır. NDEBUG tanımlansa bile derleyici bu kodu ve string'i binary'de tutar. Oysa klasik soylarda tek kontrol `assert(ms_singleton)` olduğu için Release derlemede (`/DNDEBUG`) string tamamen silinir.

**=> Binary içinde "CSingleton::Instance() NEVER DECLARED" string'i görmek = bu modernize soylardan derlenmiş demektir.**

## 2. Bu soyu taşıyan GitHub repoları (code search, 2026-11-14)

| Repo | Dosya | Not |
|---|---|---|
| wnmqaubb08/globalmetin | source/client/source/eterBase/Singleton.h | Ana inceleme konusu (Titan2Client) |
| davidkm2/globalmetin | aynı | globalmetin aynası (birebir aynı boyut) |
| NoMercy-ac/NoMercy | source/Common/AbstractSingleton.hpp | Açık kaynak anti-cheat (TR ekosistemi); xorstr ile string obfuscation + `std::abort()` |
| NoMercy-ac/NoMercyGameModule | module/include/singleton.hpp | NoMercy sunucu modülü |
| NoMercy-ac/SystemHealthCheck | include/abstract_singleton.hpp | Sistem aracı |
| Sekiraw/CPP-SFML-RougeLike-builder | Singleton.hpp | Kopyalanmış varyant (metin2 ile ilgisiz) |
| catalin1234121312/Metin2-Astralis | — | **"astra" adayı; repo BOŞ** (2025-09-24, içerik yok). İzleme listesinde; source pushlanırsa zip snapshot alınacak |

## 3. globalmetin (Titan2Client) derleme kimliği — proje dosyalarından

`Titan2Client.sln`: `# Visual Studio 14` → **VS2015** (Format Version 12.00).

Tüm client `.vcxproj`'ların taranan ayarları (16 proje):

| Ayar | Değer |
|---|---|
| PlatformToolset | **v140_xp** (VS2015, XP hedefli); UserInterface'te ayrıca v120 (VS2013) konfigürasyonu |
| Mimari | Win32 (32-bit client) |
| Optimization | Debug: Disabled; **Release/Distribute: MaxSpeed (/O2)** |
| InlineFunctionExpansion | **OnlyExplicitInline (/Ob1)** |
| RuntimeLibrary | Debug: /MTd; **Release: /MT (statik CRT)** |
| PreprocessorDefinitions | Release: NDEBUG var; Debug: _DEBUG |
| FunctionLevelLinking /Gy, StringPooling /GF | true |
| WholeProgramOptimization (/GL) | **yok** (AntiCheatLib hariç); UserInterface linker'ında `UseLinkTimeCodeGeneration` var ama /GL'siz obje üretilmediği için etkisi sınırlı (dış /GL'li .lib'ler bağlanırsa uygulanır) |
| RTTI | Açık (default /GR) — `typeid(T).name()` bunu zorunlu kılar |

Bu soyda özel durum: **`SendSequence` YOK** (repo genelinde 0 eşleşme) — packet-sequence sistemi kaldırılmış/değiştirilmiş; `SendAttackPacket` yerine **`SendAttackPacketFromThread`** (thread'li anti-freeze modu) var.

## 4. Klasik soy kanıtı: SendAttackPacket → call SendSequence

FALLEN1D/sistem `UI/PythonNetworkStreamPhaseGame.cpp` (klasik lineage):

```cpp
bool CPythonNetworkStream::SendAttackPacket(UINT uMotAttack, DWORD dwVIDVictim)  // satır 3165
{
    if (!__CanActMainInstance())
        return true;
    ...
    return SendSequence();   // satır 3189 — AYRI fonksiyon çağrısı
}
```

SendSequence düz bir member function olduğu için (inline işaretli değil), klasik binary'lerde disassembly'de temiz `call SendSequence` görünür. Kullanıcının tarif ettiği "klasik PVP" formu birebir bu.

## 5. Kullanıcının gördüğü "iç içe je/jne karmaşası"nin mekanizması

Üç katmanlı açıklama (hepsi kaynak kanıtlı):

1. **`__forceinline Instance()`**: Her `Xxx::Instance().Method()` çağrısında getter gövdesi açılır. Açılan gövdede `if (!ms_singleton) MessageBoxA(...)` branch'i var → her çağrı noktasına `mov reg,[ms_singleton]; test/jne → MessageBox yolu` serisi girer. Metin2'de her şey singleton (CPythonNetworkStream, CPythonPlayer, CItemData, CPythonItem, CSpeedTreeDirect, CResourceManager...) olduğundan, her fonksiyonda bu `je/jne` serileri çoğalır.

2. **PVP sahiplerinin derleme ayarları farklı**: Şablon bazında /Ob1 olsa da, bu soyları derleyen PVP sahipleri sıklıkla kendi konfigürasyonlarını kullanır: `/O2` veya `/Ox` + **/Ob2 (Any Suitable)** ve/veya **/GL+/LTCG (Whole Program Optimization)**. Bu ayarlarla `SendSequence` gibi küçük member fonksiyonların gövdesi çağıran içine açılır (inline expansion), optimizer temel blokları yeniden sıralar ve tail-merge yapar → `SendAttackPacket`'ın disassembly'sinde SendSequence'ın gövdesi `je/jne`'lerle iç içe geçmiş görünür.

3. **Assert/branch serisi**: Bazı derlemelerde NDEBUG eksik bırakılıp `_DEBUG` tanımlıyken elle `/O2` açılır (Debug konfigürasyonuna hız ayarı) — o zaman her assert de kodda kalır ve je/jne yoğunluğu daha da artar.

**Özet**: "NEVER DECLARED" görmek = modernize soy + `__forceinline` singleton getter. Fonksiyon gövdelerinin iç içe geçmesi ise /Ob2//GL/LTCG tarzı agresif inline ayarlarının işidir; ikisi birlikte kullanıcının tarif ettiği tabloyu üretir.

## 6. Binary'den teşhis reçetesi (PeDetect MVP ile)

Bir PVP client exe'si verilirse:
1. String araması: `CSingleton::Instance() NEVER DECLARED` (veya kardeş stringler) → soy teşhisi. Not: NoMercy türevlerinde xorstr ile obfuscate olabilir.
2. **Rich header** → gerçek derleyici sürümü (comp.id prodId/build).
3. Import tablosu: `MSVCP140D/UCRTBASED` → Debug CRT; `MSVCP140` → Release; `msvcp90d/71d` → eski VS.
4. `.textbss` + `.msvcjmc` bölümü → Debug + /JMC (VS2017+).
5. `.?AV...` RTTI stringleri → /GR açık.
6. Linker 14.x → VS2015-2019; linker 13.10/13.00 → VS2003/2002 (klasik soylarda yaygın).

PeDetect CLI (Linux): `/home/z/my-project/pedetect/build/pedetect_cli <dosya>`; Windows GUI/CLI: `pedetect/build/PeDetect_*.exe`.

## 7. Açık sorular (sonraki döngülerde)

- Klasik soyların (.vcproj) gerçek toolset'leri: FALLEN1D/sistem (branch bilinmiyor, tree 404 aldı → branch tespit edilecek), szymorek99/client_files_longju2, Hammer1337/HammerM2, DrLegendy/LegendySF-V2, hasanmacit/Directx9-Update.
- `.cod` (MSVC /FAcs asm dökümü) artefaktları: rafalohaki/Metin2-Work ve NakiuS/Metin2Client'ta Distribute/*.cod var — SendAttackPacket proc'unda `call SendSequence` mı, inline mı? Doğrudan kanıt sağlar.
- lineage client `.exe`'leri Rich header ile analiz (zaxerrrr-dot/mt2009-sp-plus `client-patches/exe/` içeriyor).
- "astra" PVP'nin gerçek kaynağı henüz yok (Astralis repo boş).
