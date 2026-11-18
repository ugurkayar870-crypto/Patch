# baglama_synth.maxpat – gen~ synth

Max 8 veya 9 ile aç. PDF'teki Bölüm 1'in (Adım 1–6) hazır hâli.

## Ana patch (açınca gördüğün)
- **NEKTAR:** `notein` → `mtof` → `sig~` → gen~ sol giriş (frekans). Hız → `/ 127.` → `sig~` → gen~ sağ giriş (gürlük). Tuşu bırakınca hız 0 olur, ses susar.
- **KLAVYESİZ TEST:** `440`'a sonra `1`'e tıkla → ses çalar. `0` → susar.
- **AYARLAR:** 5 sayı kutusu → `prepend <ad>` → `s ayar` → `r ayar` → gen~. Açılışta başlangıç değerleri yüklenir.
- **ÇIKIŞ:** `ezdac~` (tıklayınca ses açılır) ve `scope~`.
- **DIŞA AKTARMA:** gen~'i seç → Inspector → `exportfolder` = `gen_export`. Sonra `exportcode` mesajına tıkla.

## gen~ içi (gen~'e çift tıkla)
| Adım | Nesneler | Ne yapar |
|---|---|---|
| 1 Dalgalar | `phasor`, `* 2`, `- 1`, `abs`, `> 0.5`, `cycle`, `selector 4` | 4 dalga üretir, `dalga` param'ı birini seçer |
| 2 Filtre | `samplerate`, `exp`, `!- 1`, `history`, `mix` | Tek kutuplu alçak geçiren. Katsayı = 1 − e^(−2π·kesim/sr) |
| 3 Zarf | `mstosamps`, `slide` | Gürlüğü atak/bitiş süresiyle yumuşatır |
| 4 Çıkış | `*`, `*`, `clip -1 1`, `out 1` | ses × zarf × seviye |

param'lar: `dalga` (1–4), `kesim` (20–12000 Hz), `atak` / `bitis` (1–5000 ms), `seviye` (0–1).

## Kontrol et
- Max Console'da kırmızı hata var mı?
- Testere sesi parlak, sinüs yumuşak mı? `kesim`'i 200'e indirince ses boğuklaşıyor mu?
- Tezin için gen~ içinin ekran görüntüsünü al.
