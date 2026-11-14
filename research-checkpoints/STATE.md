# STATE — Metin2 "CSingleton::Instance() NEVER DECLARED" araştırması

Bu dosya periyodik checkpoint döngüsünün (cron, her 5 dk) ana kontrol dosyasıdır.
Her turda: ROADMAP'teki ilk `[ ]` adımını uygula → bulguları ilgili research/*.md'ye EKLE →
NEXT STEP satırını güncelle → `bash /home/z/my-project/scripts/checkpoint_run.sh` çalıştır → kısa özet ver.

NEXT STEP: S5 - "astra" PVP kaynak araması: repo search (astra2 metin2, astra client source), code search ("astra" metin2), WebSearch metin2.dev/m2dev forumları → research/ASTRA_HUNT.md

## ROADMAP

- [x] S1 (2026-11-14): Soy kimliği + globalmetin derleme analizi + klasik SendSequence kanıtı → research/FINDINGS_01_globalmetin_lineage.md
- [x] S2 (2026-11-14): Toolset tablosu + Singleton varyant karşılaştırması (önceki oturum artefaktları işlendi) → research/FINDINGS_01b_artifacts_evidence.md (artifacts/ klasörü repoda)
- [x] S3a (2026-11-14): SendAttackPacket proc asm analizi: temiz `call ?SendSequence@CNetworkStream@@QAE_NXZ`; compiler 18.00.40629 (VS2013 Upd5) → FINDINGS_01b
- [x] S3b (2026-11-14): nak_PhaseGame.cod tam tarama: 240 proc, SendSequence TU'da değil, 53 call SendSequence, 4 istisna (SendSpecial/SyncPosition/ExchangeItemDel/DragonSoulRefine); binary teşhis kuralı → research/FINDINGS_02_cod.md + cod_scan_result.json (araç: scripts/cod_scan.py)
- [x] S4 (2026-11-14): Gerçek PVP client binary analizi (rafalohaki/Metin2-Work): MAIN=VS2019 (linker 14.28, 2021, Kraizy edition, PYTHON27, /GS+RTTI+/MT, Release), CONFIG=VS6 (2006); MAIN'de "NEVER DECLARED" YOK ama CSingleton RTTI descriptor'ları VAR → assert-only soy → research/FINDINGS_03_binaries.md + pedetect_*.json
- [ ] S5: "astra" PVP kaynak araması devam (repo search: astra2 / astra metin2; metin2.dev forum WebSearch) → research/ASTRA_HUNT.md
- [ ] S6: FINAL_REPORT.md sentezi: "NEVER DECLARED binary teşhis + hangi ayarla üretilir + repro vcxproj ayarları"

## KURALLAR

1. GitHub token: /home/z/my-project/.secrets/github_token — içeriğini ASLA ekrana/yazılıma/zip'e yazma; sadece Authorization header'da ve push URL'sinde kullan (push çıktısı token içerebilir → sed ile redact et).
2. Her turda SADECE BİR roadmap adımı; büyük indirme yapma (>100MB tekil dosya indirme).
3. FINDINGS_01'i overwrite etme; yeni bilgiyi mevcut dosyaya ekle veya yeni dosya aç.
4. monitor.json'a araştırma sırasında önemli bulunan repolar eklenebilir (zip izleme otomatik).
5. Araç: curl + api.github.com (token ile, rate limit 5000/h). Code search: en fazla 10 istek/dk, aralarına sleep 3 koy.
6. Her tur SONU: scripts/checkpoint_run.sh çalıştırılmadan bitirme.
7. Araştırma tamamen bitince: ROADMAP hepsi [x] → NEXT STEP: "bekleme - arastirma tamamlandi" yaz; döngü sadece monitor+push yapar.
