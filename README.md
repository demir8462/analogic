<div align="center">

# ⚡ ANALOGIC

### Olay tabanlı 10 bit SAR ADC · sky130 · uyurken nanowatt, uyanınca 36 pJ

![PDK](https://img.shields.io/badge/PDK-SkyWater%20sky130A-6f42c1)
![ADC](https://img.shields.io/badge/SAR-10%20bit%20%C2%B7%201%20MS%2Fs-orange)
![Enerji](https://img.shields.io/badge/d%C3%B6n%C3%BC%C5%9F%C3%BCm-36%20pJ-blue)
![Boşta](https://img.shields.io/badge/bo%C5%9Fta-24%20nW-2ea44f)
![Komparatör](https://img.shields.io/badge/komparat%C3%B6r-rail--to--rail-red)

</div>

---

## 🎯 Ne yapıyoruz?

Sinyal değişmediği sürece **uyuyan**, bir olay gelince kendini uyandırıp dönüşümü **kendi saatini üreterek** yapan, sonra tekrar uykuya dönen bir **10 bit SAR ADC**. Saat ağacı yok, boşta neredeyse sıfır güç.

**Neden önemli?** Sinir sinyali kaydı, titreşim izleme, olay tabanlı görüntüleme gibi uygulamalarda sinyal çoğu zaman sessizdir. Klasik ADC sessizlikte de durmadan örnekler ve güç yakar; bizimki sadece iş olduğunda çalışır.

## 🏆 Öne çıkanlar

| | |
|---|---|
| ⚡ **Dönüşüm başı enerji** | **36 pJ** (aynı PDK'daki açık kaynak rakipten ~13 kat düşük) |
| 😴 **Boşta güç** | **24 nW** (25 °C), 65 nW (75 °C) |
| 🎚️ **Rail-to-rail komparatör** | Giriş ortak modu 30 mV … 1790 mV, kalibrasyonsuz ofset **σ 1,6 – 2,8 mV** |
| ✅ **Doğruluk** | 30 … 1770 mV aralığında, tt/ss/ff köşelerinde ve 75 °C'de **−1 … 0 LSB** |
| 🧩 **Kendi saatini üretir** | Tek START darbesiyle 10 biti asenkron çözer, 123 ns |
| 🪟 **Akıllı modlar** | Erken durma (6 bit, 26 pJ), tek kararlık pencere kontrolü (9,7 pJ) |
| 🧪 **Layout'a kadar** | Komparatör çizildi, DRC/LVS temiz, çıkarılmış layout üzerinde Monte Carlo |

<div align="center">
<img src="img/guc_olay_hizi.png" width="760" alt="Güç - olay hızı">
<br><sub><b>Ortalama güç olay hızıyla ölçekleniyor:</b> saniyede birkaç olayda ADC 25 nW civarında kalıyor</sub>
</div>

<div align="center">
<img src="img/sar_donusum.png" width="760" alt="SAR dönüşümü">
<br><sub><b>Asenkron 10 bit dönüşüm:</b> her READY darbesi bir bit</sub>
</div>

## 🧱 Sistemin parçaları

Her blok önce tek başına doğrulandı, sonra bir üst seviyeye eklendi. Tasarım detayları paylaşılmıyor, sadece ne yaptıkları ve sonuçları.

### 🎚️ Komparatör
Kapasite dizisinin çıkışını referansla karşılaştırır, kararı verince kendi READY sinyalini üretir.

| Metrik | Sonuç |
|---|---|
| Giriş ortak modu | **Rail-to-rail**, 30 … 1790 mV |
| Ofset (layout, Monte Carlo) | **σ 1,6 – 2,8 mV**, kalibrasyonsuz |
| Karar süresi | 2,4 – 4,1 ns |
| Doğrulama | Layout, DRC/LVS temiz; çıkarılmış layout'ta 720/720 doğru karar (±1 mV, tüm köşeler) |

### 🧮 CDAC (kapasite DAC'ı)
Örneklenen girişi tutar ve her adımda bir biti dener; SAR'ın "terazisi".

| Metrik | Sonuç |
|---|---|
| Yapı | 10 bit ikili ağırlıklı dizi + dummy halka (ilk sürüm 9 bit) |
| Doğruluk | El ile sürülen SAR testinde doğru kod; en büyük hata **0,34 mV (< 0,1 LSB)** |
| En kötü bit oturması | **24 ns** (0,25 LSB'ye) |
| İyileştirme | Parazitik kaynaklı %1,4 kazanç hatası tespit edilip 10 kat düşürüldü |
| Layout | 16×32 2D ortak merkezli dizi, 512 aktif + 100 dummy birim, **DRC 0** |
| Layout (RC çıkarımlı) | Doğru kod; kazanç 0,984, **INL en kötü 0,07 LSB**, en kötü oturma **12 ns** |
| 10 bit layout | 130,5×151 µm, **DRC 0, LVS eş**; RC çıkarımlı testte kod 17 doğru, en kötü oturma **7,0 ns**, kazanç 0,981, INL en kötü 0,56 LSB. **Açık konu:** DNL en kötü −0,82 LSB (511→512), sebebi hat–üst plaka paraziti, düzeltme üzerinde çalışılıyor |

### 🔌 Örnekleme ve analog anahtarlar
Girişi kapasitelere alır, plakaları referanslara bağlar.

| Metrik | Sonuç |
|---|---|
| Örnekleme anahtarı | Bootstrap: direnci giriş geriliminden bağımsız, raydan raya giriş |
| Plaka anahtarları (v1) | 2 pF yükte 10 bit yarım-LSB'ye 43 ns'de oturma; layout DRC/LVS temiz, layout = şema |
| Plaka anahtarları (v2, düşük eşikli) | Oturma **13,6 ns** (3 kat hızlı), kapalı kaçak en kötü **0,31 nA** |

### 📍 Ortak mod referansı (VDD/2)
Karşılaştırma noktasını üretir; tamamen pasif ve çok düşük akımlı.

| Metrik | Sonuç |
|---|---|
| Çıkış | **0,900 V** (VDD/2, beslemeyle oransal) |
| Akım / güç | ~20 nA / ~36 nW (zayıf kol) |
| 10 bit örnekleme yükü | Güçlendirilmiş sürüm (zayıf kol + başlangıçta devreye giren güçlü kol); 100 ardışık örnekte **birikme yok**, örnek sonu VCM 0,84 / 0,96 V (tt) |
| Layout | 24,7×49,9 µm, **DRC 0, LVS eş**, RC çıkarımlı sonuç şemayla aynı |

### 🧠 SAR mantığı ve sıralayıcı
Bitleri yazar, komparatörü tetikler ve durdurur; saati kendisi üretir.

| Metrik | Sonuç |
|---|---|
| Dönüşüm | Tek START darbesiyle 10 bit, **123 ns** |
| Modlar | Erken durma (6 bit, 26 pJ), tek kararlık pencere kontrolü (9,7 pJ) |
| Güvenlik | Zaman aşımı koruması, iş bitince kendiliğinden uykuya dönüş |

## 🆕 Son güncellemeler (Ekim 2026)

**🧮 CDAC layout'u çizildi**
- 9 bit, sky130 MIM birim kapasite ~9,3 fF, toplam ~4,76 pF, ikili ölçekli sürücüler
- 16×32 2D ortak merkezli yerleşim, 512 aktif + 100 dummy birim, **DRC 0**
- RC çıkarımlı layout testi: 30 mV girişte doğru kod, kazanç 0,984, **INL en kötü 0,07 LSB**, en kötü bit oturması **12 ns**

**🔌 Analog anahtar (T-gate)**
- v1: layout DRC 0, LVS eş; layout sonucu şemayla aynı (2 pF'a 42,6 ns oturma)
- v2 (düşük eşikli): oturma **13,6 ns**; tepe açık direnç tt/ss/ff **1,6k / 2,4k / 1,2k Ω** (v1: 5,2k / 15,8k / 2,2k Ω); kapalı kaçak en kötü **0,31 nA**

| Anahtar | Oturma (2 pF) | Tepe Ron tt / ss / ff | Layout |
|---|---|---|---|
| v1 | 42,6 ns | 5,2k / 15,8k / 2,2k Ω | DRC 0, LVS eş |
| **v2 (LVT)** | **13,6 ns** | **1,6k / 2,4k / 1,2k Ω** | DRC 0, LVS eş |

**🧮 CDAC 10 bit layout:** 130,5×151 µm, DRC 0, LVS eş; RC çıkarımlı testte kod 17 doğru, en kötü oturma **7,0 ns**. Açık konu: DNL −0,82 LSB, düzeltme sürüyor

**📍 Ortak mod referansı (v3):** köşelerde **0,900 V**, boşta **36 nW** (25 °C); 10 bit yükle 100 örnekte birikme yok; layout 24,7×49,9 µm, DRC 0, LVS eş

**🔧 Masaüstü demo (yolda):** 8 bit R-2R SAR + örnekle-tut, SAR mantığı Nexys A7 FPGA üzerinde; parçalar sipariş edildi

## 🏁 Literatürde neredeyiz?

Bizim sayılar **simülasyon** (sky130, 1,8 V, tt, 25 °C; komparatör sayıları çıkarılmış layout'tan). Rakip sayıları yayınlanmış çalışmalardan alındı, isim verilmedi.

**SAR ADC**

| | **Biz (SAR v3, sim)** | Rakip A: açık kaynak sky130 SAR | Rakip B: 180 nm SAR (silikon) |
|---|---|---|---|
| Teknoloji / besleme | sky130, 1,8 V | sky130, 1,8 V | 180 nm, 1,8 V |
| Çözünürlük | 10 bit | 12 bit (ikili olmayan) | 12 bit |
| Örnekleme hızı | 1 MS/s (dönüşüm 123 ns) | 1,44 MS/s | 2 MS/s |
| Güç (tam hızda) | **36 µW** | 703 µW | 183 µW |
| Dönüşüm başı enerji | **36 pJ** | ~488 pJ | ~92 pJ |
| Walden FoM | ~43 fJ/adım (ENOB 9,7 varsayımı) | yayında yok | 48,6 fJ/adım (ölçülmüş) |
| Boşta güç (olay beklerken) | **24 nW** (75 °C'de 65 nW) | sürekli çalışıyor | sürekli çalışıyor |
| Alan | henüz yok | 0,175 mm² | 0,36 mm² |

**Komparatör**

| | **Biz (v6b, layout + MC)** | Rakip C: StrongARM (45 nm, sim) |
|---|---|---|
| Giriş aralığı | **Rail-to-rail** (VCM 30 … 1790 mV) | sabit VCM civarı |
| Ofset | **σ 1,6 – 2,8 mV** (Monte Carlo) | 28 mV |
| Karar süresi | 2,4 – 4,1 ns (CLK → READY) | 63 – 136 ps |
| Besleme | 1,8 V | 1,2 V |

**Kısaca:** olay tabanlı çalışma sayesinde boşta rakiplerin binde biri kadar güç, dönüşüm başına da çok daha az enerji. Rakip B silikonda ölçülmüş bir çip; bizim sayılarımız henüz simülasyon, ENOB doğrulaması ve tam sistem layout'u yolda.

## 🗺️ Yol haritası

- [x] Rail-to-rail StrongARM komparatör: layout, DRC/LVS, Monte Carlo
- [x] Olay tabanlı 10 bit SAR referans tasarımı: 36 pJ, boşta 24 nW
- [x] CDAC, örnekleme anahtarları ve ortak mod referansı blok blok doğrulandı
- [x] CDAC (10 bit) ve analog anahtar layout'ları (DRC temiz, LVS eş, RC çıkarımlı doğrulama)
- [x] Ortak mod referansı v3: layout, 10 bit yükle çok örnekli doğrulama
- [ ] CDAC DNL iyileştirmesi (−0,82 LSB)
- [ ] Masaüstü demo: 8 bit R-2R SAR, Nexys A7 üzerinde
- [ ] Tüm sistemi hiyerarşik şemada birleştirmek
- [ ] FFT ile ENOB doğrulaması
- [ ] Tam ADC layout'u ve FPGA (Nexys A7) üzerinde SAR mantığı denemesi
- [ ] Piksel/kanal seviyesinde olay algılama entegrasyonu

## 🛠️ Araçlar

SkyWater **sky130A** · xschem · ngspice · magic · netgen · IIC-OSIC-TOOLS

---

<div align="center"><sub>Tasarım dosyaları şimdilik açık kaynak değil. İş birliği için iletişime geçebilirsiniz.</sub></div>
