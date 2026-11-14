# research-checkpoints

Metin2 PVP araştırması: **"CSingleton::Instance() NEVER DECLARED"** string'li sunucuların derleyici kimliği
ve fonksiyon iç içe geçmesi (inlining) fenomeni. + PeDetect MVP (PE derleyici tespit aracı) kullanım bulguları.

Bu klasör periyodik olarak otomatik checkpoint'lenir:
- `CHECKPOINT_STATUS.txt` → en son checkpoint no + özet + izlenen repo snapshot durumu
- `STATE.md` → araştırma yol haritası ve güncel adım
- `research/` → bulgu dosyaları (FINDINGS_*, REPOS_TABLE, ASTRA_HUNT, FINAL_REPORT...)
- `monitor.json` → zip snapshot izlenen GitHub repoları

İzlenen repolarda (ör. globalmetin) yeni commit görünce kaynak kod zip olarak
`download/source_snapshots/` altına kaydedilir ve buraya not edilir.
