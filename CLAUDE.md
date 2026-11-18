# Bağlama Synth – Teensy 4.1 projesi

Bu dosya Claude Code için proje özetidir. Kullanıcı: Uğur, İnönü Üniversitesi Müzik Teknolojisi öğrencisi. Proje bitirme tezi için yapılıyor.

## Kullanıcıyla çalışma kuralları

- Uğur ile **Türkçe** konuş. **Kısa ve sade** anlat; uzun açıklamaları okumakta zorlanıyor. C++ bilgisi az, Max/MSP biliyor. Yeri geldikçe Max'teki karşılığıyla açıkla (örnek: `AudioConnection` = Max'teki patch kablosu).
- Görevin hem kodu yazmak hem de **Uğur'a anlatmak**: her adımda ne yaptığını ve neden yaptığını tezde kullanabileceği kadar açık ama kısa söyle.
- Uğur'un elinde `baglama_synth_yapilacaklar.pdf` var; yaptığı her şey orada sırayla yazıyor (aşağıda "Uğur'un PDF'i"). O PDF'teki bir adımı sorarsa ona göre cevap ver. Teensy henüz gelmediyse önce PDF'in Bölüm 1'inde (gen~) yardım et.
- Kod yorumlarını **Türkçe** yaz. Her bloğun ne yaptığını tezde anlatabileceği açıklıkta açıkla.
- İşi aşağıdaki **aşamalara** böl. Her aşama ayrı bir sketch klasöründe olsun (örnek: `asama1_ses_cikisi/asama1_ses_cikisi.ino`).
- Her aşamanın sonunda Uğur'a şunu söyle: kodu nasıl yükleyecek, neyi nasıl deneyecek, ne duyması ya da görmesi gerekiyor. **Uğur testin sonucunu yazmadan sonraki aşamaya geçme.**
- Mümkünse kodu `arduino-cli` ve Teensy çekirdeğiyle derleyip hataları kendin düzelt. Karta yüklemeyi Uğur Arduino IDE'den yapar.
- Listede olmayan yeni bir parça önermeden önce sor. Bütçe ve temin süresi kısıtlı.
- Hoca, projenin bir bilgisayarla (Raspberry Pi gibi) değil, mikrodenetleyiciyle yapılmasını istiyor. Pure Data kullanılmıyor.
- **Max/MSP kullanılıyor:** Synth sesi Uğur tarafından Max'in **gen~** bölümünde tasarlanıyor, C++ olarak dışa aktarılıp Teensy'ye taşınacak (aşağıda "Max gen~ ile ses motoru"). Bu tezin ve yüksek lisans başvurusunun önemli bir parçası; gen~ yamasını Uğur PDF'e bakarak kendisi kuruyor. Sorduğunda nesneleri kısaca açıkla, takıldığı yeri birlikte bul; yamayı onun yerine yazma.

## Projenin amacı

Bir çalgının (öncelikle bağlama) sesini alıp başka bir çalgının sesine dönüştüren, isteğe bağlı olarak MIDI klavyeyle de çalınabilen, bilgisayarsız çalışan bir synthesizer.

1. **Çalgı modu:** Gelen sesin perdesi (Hz) ve gürlüğü ölçülür. Bu perdede ve gürlükte seçili ses çalınır: synth sesi (testere dalgası + filtre) ya da kayıtlı bir çalgı sesi (zurna, ney vb.).
2. **Klavye modu:** Nektar Impact LX88+ USB MIDI klavyeden gelen notalar aynı ses motorlarını çalar.
3. **Önemli tez noktası:** Çalgı modunda perde nota adına yuvarlanmaz; ölçülen frekans doğrudan kullanılır. Böylece bağlamanın komaları korunur. Klavye modu doğal olarak 12 tonludur.

## Donanım

| Parça | Not |
|---|---|
| Teensy 4.1 | Arduino IDE + Teensyduino ile programlanır |
| PCM5102A I2S DAC modülü (mor, GY-PCM5102) | Ses çıkışı; üzerinde 3,5 mm jak var |
| Behringer UMC22 | Analog ön amfi olarak kullanılıyor (Teensy'ye USB ile bağlı değil). Piezo onun enstrüman girişine takılı, Direct Monitor açık. Bir çıkışı Teensy'nin analog girişine, diğer çıkışı (ham ses) doğrudan hoparlöre gider |
| Piezo disk manyetik | Bağlamaya yapıştırılmış; ucu kesilmiş jaklı enstrüman kablosuyla UMC22'ye |
| Nektar Impact LX88+ | Sürücüsüz (class-compliant) USB MIDI klavye; 8 pot, 9 fader, 8 pad |
| microSD kart (32 GB) | Teensy 4.1'in kendi kart yuvasında (`BUILTIN_SDCARD`); çalgı kayıtları için |
| Kesilmiş USB uzatma kablosu | Dişi ucu Nektar için; telleri Teensy'nin USB host pinlerine lehimli |
| Breadboard seti (yarım boy) | 10 kΩ ve 1 kΩ dirençler, 6 buton ve LED'ler bu setten |
| 10 µF elektrolitik kondansatör | Analog giriş devresi için |
| 2 portlu 5V USB şarj adaptörü | Teensy ve UMC22'yi besler |

PSRAM yok; Teensy Audio Shield yok.

## Bağlantılar

| Teensy 4.1 | Nereye |
|---|---|
| VIN (5V) | PCM5102A VIN |
| GND | PCM5102A GND |
| Pin 21 (BCLK1) | PCM5102A BCK |
| Pin 20 (LRCLK1) | PCM5102A LCK |
| Pin 7 (OUT1A) | PCM5102A DIN |
| GND | PCM5102A SCK (dahili PLL kullanılsın diye) |
| 3.3V | 10 kΩ → A2 |
| GND | 10 kΩ → A2 |
| Pin 16 (A2) | İki 10 kΩ direncin birleştiği nokta; buradan 1 kΩ koruma direnci → kondansatörün **+** bacağı |
| — | Kondansatörün **−** bacağı → UMC22'den gelen jak kablosunun uç (tip) teli; kablonun gövde (sleeve) teli → GND |
| USB host pinleri (5V, D−, D+, GND) | USB kablosunun kırmızı, beyaz, yeşil, siyah telleri (renkleri multimetreyle doğrulat) |
| Pin 2, 3, 4, 5 | Butonlar; butonun diğer bacağı GND'ye (`INPUT_PULLUP`) |
| Pin 13 | Kart üzerindeki LED, durum göstergesi |

PCM5102A modülünün arka yüzündeki lehim köprüleri için yaygın ayar: 1 = L (FLT), 2 = L (DEMP), 3 = H (XSMT), 4 = L (FMT). Modülden ses gelmezse önce bunu kontrol ettir; emin olmak için Uğur'dan modülün arka yüzünün fotoğrafını iste.

**Güvenlik:** Teensy 4.1 pinleri 5V'a dayanıklı değil; analog girişe 0–3.3V dışında gerilim gelmemeli. UMC22'nin çıkış seviyesini düşük-orta tutmasını hatırlat. Hoparlör hiçbir zaman doğrudan Teensy pinlerine bağlanmaz. Lehim işlerini okuldaki hoca yapabilir; Uğur'a lehim gereken her yeri açıkça listele.

## Yazılım

- Arduino IDE 2 + Teensyduino. Kart: Teensy 4.1. USB Type: Serial.
- Teensy Audio Library (Teensyduino ile gelir). Faydalı nesneler: `AudioInputAnalog` (A2), `AudioAnalyzeNoteFrequency` (perde, YIN algoritması), `AudioAnalyzePeak` ya da `AudioAnalyzeRMS` (gürlük), `AudioSynthWaveform` / `AudioSynthWaveformModulated`, `AudioFilterStateVariable`, `AudioEffectEnvelope`, `AudioMixer4`, `AudioOutputI2S`.
- Ses bağlantılarını göstermek için PJRC'nin tarayıcıdaki Audio System Design Tool'unu öner; Max'e benzediği için Uğur'un işine yarar.
- MIDI: `USBHost_t36` (`USBHost`, `USBHub`, `MIDIDevice`; sorun çıkarsa `MIDIDevice_BigBuffer`).
- Butonlar: Teensyduino ile gelen `Bounce` kütüphanesi.
- Değişken hızlı sample çalma için önce `newdigate/teensy-variable-playback` kütüphanesini değerlendir. Uygun değilse kısa kayıtları RAM'e yükleyip basit bir resampler yaz. PSRAM olmadığını unutma.

## Max gen~ ile ses motoru

Akış: **Max (gen~ patch) → `exportcode` ile C++ dosyaları → Arduino IDE'deki sketch klasörü → Teensy.**

- **gen~ yamasının tasarımı (PDF'teki ile aynı):**
  - `in 1` = frekans (Hz), `in 2` = gürlük (0–1; 0 olunca susar), `out 1` = ses.
  - Dalgalar: `phasor` → testere (`* 2`, `- 1`), testereden üçgen (`abs`, `* 2`, `- 1`), kare (`> 0.5`, `* 2`, `- 1`), sinüs (`cycle`). `selector 4` ile seçilir.
  - Filtre: tek kutuplu alçak geçiren; `history` + `mix`, katsayı = 1 − e^(−2π·kesim/samplerate). Rezonans yok.
  - Zarf: `in 2` → `slide` (atak ve bitiş `mstosamps` ile örnek sayısına çevrilir).
  - Çıkış: filtreli ses × zarf × seviye → `clip -1 1` → `out 1`.
- **param adları (Teensy kodunda bu adları kullan):**

  | param | Başlangıç | Aralık | Anlamı |
  |---|---|---|---|
  | `dalga` | 1 | 1–4 | 1 testere, 2 üçgen, 3 kare, 4 sinüs |
  | `kesim` | 3000 | 20–12000 Hz | filtre kesim frekansı |
  | `atak` | 20 | 1–5000 ms | açılma süresi |
  | `bitis` | 300 | 1–5000 ms | kapanma süresi |
  | `seviye` | 0.8 | 0–1 | ana ses seviyesi |

- **Dışa aktarma:** Uğur gen~'e `@exportname baglama_synth` verdi; Inspector'dan `exportfolder` olarak `gen_export` klasörünü seçip `exportcode` mesajına tıklıyor. Bu klasörü proje klasörüne koyacak; içinde `baglama_synth.cpp`, `baglama_synth.h` ve `gen_dsp` klasörü olmalı (adlar Max sürümüne göre biraz farklı olabilir; klasöre bakıp doğrula).
- **Senin yapacağın:** Dışa aktarılan kodu saran bir Teensy ses nesnesi yaz (`AudioStream`'den türeyen, örnek ad `AudioGenSynth`; PJRC'nin "Creating New Audio Objects" sayfasındaki yapıyla). Her blokta (128 örnek, 44,1 kHz) gen~'in `perform` fonksiyonunu çağırsın; `param` değerlerini `setparameter` ile Nektar potlarından ve perde takibinden ayarlasın. Derleme hatalarını sen çöz; Teensy için genlib'in 32 bit float ayarını (`GENLIB_USE_FLOAT32` gibi) değerlendir.
- **gen~'de olmayanlar:** Perde takibi (`fzero~` gen~'de yok), Nektar, butonlar, ses girişi/çıkışı ve SD kart Teensy kütüphanesiyle yapılır. gen~ sadece sesi üretir.
- **Bellek:** gen~ patch'inde büyük `data`/`buffer` kullanılmasın; PSRAM yok.
- gen~ yolu takılırsa yedek çözüm: synth'i Teensy Audio Library nesneleriyle (Audio System Design Tool) kurmak. Bunu Uğur'a sormadan uygulama.

## Aşamalar

**Aşama 0 – Kurulum.** Arduino IDE ve Teensyduino kurulumunu anlat. Blink ile kartın çalıştığını doğrulat.

**Aşama 1 – Ses çıkışı.** 440 Hz sinüs → `AudioOutputI2S` → PCM5102A. Kulaklıkla dinletilir. Ses yoksa lehim köprülerini ve bağlantıları kontrol ettir.

**Aşama 2 – Nektar.** USB host ile gelen nota ve CC mesajlarını Serial Monitor'e yazdır; her pot ve faderin CC numarasını Uğur'a not ettir. Sonra basit bir synth: testere dalgası + filtre + zarf, 4 ses (polifonik). Potlar kesim frekansını, atak ve bitiş süresini ve ses seviyesini kontrol etsin; pad 1–4 dalga seçimi (gen~ param'larıyla aynı eşleme). Bu ilk synth, Teensy kütüphanesiyle yapılan geçici bir test sürümüdür.

**Aşama 2b – gen~ synth'i Teensy'ye taşıma.** Uğur'a gen~ patch'ini kurdur ve dışa aktart; sen `AudioGenSynth` sarmalayıcısını yaz. Aşama 2'deki geçici synth'in yerine gen~ synth'i Nektar'la çalsın. Sonraki aşamalarda synth sesi olarak gen~ kullanılır.

**Aşama 3 – Analog giriş testi (riskli aşama).** `AudioInputAnalog(A2)` → `AudioAnalyzePeak` → Serial'a seviye yazdır, sonra sesi doğrudan çıkışa ver. `AudioInputAnalog`'un Teensy 4.1'de beklendiği gibi çalışıp çalışmadığı belirsiz; çalışmazsa durumu Uğur'a açıkça anlat ve seçenekleri sun (Teensyduino'daki ADC kütüphanesiyle kendi giriş nesnesini yazmak ya da yurt dışından Teensy 4 Audio Shield Rev D almak).

**Aşama 4 – Perde takibi.** `AudioAnalyzeNoteFrequency` ile ölçülen Hz ve olasılık değerini Serial'a yazdır. Önce bir test tonuyla (telefondan ton üretip UMC22'ye vererek), sonra bağlamayla denet. Eşik ve gecikme ayarlarını açıkla.

**Aşama 5 – Çalgı modu synth.** Ölçülen perde → gen~ synth'in frekans girişi (yumuşatarak, yuvarlamadan). Gürlük → ses seviyesi; eşiğin altında sessizlik (gürültü kapısı). 50–2000 Hz dışındaki ölçümleri yok say.

**Aşama 6 – Sample motoru.** SD karttaki WAV kayıtlarını (`zurna.wav`, `ney.wav`…) seçilen perdeye kaydırarak çal: hız = ölçülen perde ÷ kaydın perdesi. Döngü ile uzun notalar kesilmesin. Butonlarla çalgı seçimi.

**Aşama 7 – Birleştirme.** Tek sketch: mod butonu (çalgı/klavye), çalgı seçimi, LED göstergesi, Nektar potlarıyla parametreler. Gecikmeyi ölçme yöntemini anlat (ham ses ve işlenmiş ses aynı anda kaydedilip aradaki fark ölçülür).

**Aşama 8 – İsteğe bağlı.** Ayarları kaydetme, kutulama önerileri.

## Uğur'un PDF'i

`baglama_synth_yapilacaklar.pdf` içindekiler (Uğur bu sırayla ilerliyor):

- **Bölüm 1 – Max'te gen~ synth:** Adım 1 dalgalar, Adım 2 filtre, Adım 3 zarf, Adım 4 çıkış, Adım 5 Max'te Nektar ile test (`notein` → `mtof` → `sig~` → gen~ sol giriş; hız `/ 127.` → `sig~` → sağ giriş; ayarlar `prepend <ad>` → `s ayar` / `r ayar` → gen~), Adım 6 dışa aktarma.
- **Bölüm 2 – Donanım:** UMC22'yi bilgisayarsız deneme, lehim listesi, breadboard bağlantıları (bu dosyadaki tabloyla aynı).
- **Bölüm 3 – Teensy ve Claude Code:** Arduino IDE 2 + Teensy desteği (Boards Manager URL: `https://www.pjrc.com/teensy/package_teensy_index.json`), seninle çalışma, aşama tablosu (0–7).
- **Bölüm 4 – Çalgı kayıtları:** mono WAV, 44,1 kHz, 16 bit, kısa adlar (`zurna.wav`), her kaydın perdesi not edilir, microSD (FAT32).
- **Bölüm 5 – Tez:** gecikme, perde doğruluğu (komalı perdeler) ve dinleme testi ölçümleri.

## Tez için

Her aşamanın sonunda kısa bir özet yaz: ne yapıldı, hangi Audio Library nesneleri kullanıldı, neden. Uğur bunları teze ekleyecek.

Tezde vurgulanacak akış: ses Max gen~'de tasarlandı → C++'a çevrildi → Teensy'de bilgisayarsız çalışıyor. gen~ patch'inin ekran görüntülerini ve dışa aktarma adımlarını kaydetmesini hatırlat.
