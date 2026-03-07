# Hybrid-ZeroShot-Anomaly-Detection

> 🌐 **[English Version (İngilizce)](README_EN.md)** | **Türkçe** ✓

Bu proje, **CLIP** ve **DINOv2** modellerini kullanarak üretim/sanayi verilerinde (örn. MVTec AD veri seti) sıfır-örneklem (zero-shot) anomali tespiti yapmayı amaçlayan melez (hibrit) bir yaklaşım sunmaktadır. Bu çalışma, konuyla ilgili akademik bir makale için hazırlanmış olup kodun tam ve çalıştırılabilir halini içermektedir.

## Yöntemler ve Yaklaşımlar

Proje, üç farklı anomali tespit yaklaşımını analiz etmekte ve karşılaştırmaktadır:

1. **Hybrid (Hibrit) Yaklaşım**: 
   - **CLIP** modeli kullanılarak görüntüler ön taramadan geçirilir (image-level filtreleme). Bu aşama oldukça hızlıdır.
   - Sadece şüpheli ("flagged") olarak işaretlenen görüntüler **DINOv2** modeline gönderilerek detaylı piksel bazlı (pixel-level) lokalizasyon analizi yapılır.
   - Bu yöntem sayeside hem hız hem de yüksek detaylı lokalizasyon başarısı elde edilir.
   
2. **CLIP-only Yaklaşım**:
   - Sadece CLIP modelini kullanarak görüntü düzeyinde (image-level) anomali tespiti yapılır. Lokalizasyon sağlamaz ancak hızlı çalışır.
   
3. **DINO-Max Yaklaşımı**:
   - Herhangi bir ön filtreleme yapmaksızın sadece DINOv2 modeli kullanılarak piksel düzeyinde anomali tespiti ve lokalizasyon yapılır.
   - **Görüntü-seviye skorlaması** literatür standardı olan **max aggregation** (patch anomaly map'ten maksimum değer) ile hesaplanır.
   - En yüksek hesaplama maliyetine sahiptir fakat en detaylı analizi sunar.

## Kullanılan Veri Seti (MVTec AD)

Bu çalışmada, endüstriyel görsel hata tespiti (industrial visual anomaly detection) alanında standart ve yaygın olarak kullanılan halka açık bir veri seti olan **MVTec AD** (Anomaly Detection) kullanılmıştır. 
Veri setini incelemek ve indirmek için resmi bağlantıyı ziyaret edebilirsiniz:  
🔗 [MVTec AD Dataset Official Page](https://www.mvtec.com/company/research/datasets/mvtec-ad)

## Motivasyon ve Araştırma Kapsamı (Scope)

**Önemli Not:** Bu çalışmanın birincil amacı, mevcut literatürdeki en yüksek (State-of-the-Art / SOTA) anomali tespit skorunu elde etmek **değildir.** 
Ana motivasyonumuz; CLIP gibi görüntü bazında (image-level) oldukça hızlı çalışan ancak lokalizasyon yeteneği olmayan bir model ile, DINOv2 gibi piksel bazında (pixel-level) çok detaylı ve başarılı lokalizasyon yapabilen ancak hesaplama maliyeti yüksek bir modeli **hibrit (melez)** bir yapıda birleştirmektir. 
Bu "ön-filtreleme" (gating) konsepti sayesinde, endüstriyel üretim hatlarında saniyeler içinde binlerce normal ürünü düşük donanım maliyetiyle eleyip, sadece şüpheli ("flagged") ürünleri ağır anomali haritalandırmasına (DINOv2) sokarak **hız ve doğruluk (işlem maliyeti) arasında optimum bir denge** kurulabileceği kanıtlanmaktadır.

## Teknik Detaylar

### DINOv2 Görüntü-Seviye Skorlaması
Bu çalışmada, literatür standardı olan **max aggregation** yöntemi kullanılmaktadır. DINOv2 her görüntü için patch-level anomaly map üretir ve görüntü-seviye skorunu bu map'teki **maksimum değer** olarak hesaplar:

```python
# Literatür standardı
image_score = np.max(anomaly_map)
```

Bu yaklaşım, WinCLIP, PatchCore ve diğer patch-based anomaly detection yöntemlerinde yaygın olarak kullanılmaktadır.

### Mesafe Metriği
k-NN (k-Nearest Neighbors) işlemlerinde **cosine distance** metriği kullanılmıştır:

```python
nn = NearestNeighbors(n_neighbors=k, algorithm="auto", metric="cosine")
```

Feature'lar L2 normalize edildikten sonra cosine distance ile karşılaştırılır, bu da literatürde standart bir uygulamadır.

## Kullanılan Teknolojiler ve Tekrarlanabilirlik

- **OpenAI CLIP**: Görüntü bazlı genel anomali özelliklerini çıkarmak ve filtrelemek için.
- **Meta DINOv2**: Piksel bazlı haritalama (patch-level feature extraction), KNN indeksleme ve anomali ısı haritaları (heatmap) oluşturmak için.
- **Python Kütüphaneleri**: `pandas`, `opencv-python`, `scikit-learn`, `ftfy`, `regex`, ve veri yönetimi için PyTorch altyapısı.
- **Tekrarlanabilirlik (Reproducibility) ve Cihaz Ayarları**: Deneylerin tutarlı ve akademik anlamda tekrarlanabilir olması için rastgelelik tohumu (seed) `SEED = 42` olarak (Numpy, Random ve PyTorch CPU/GPU fonksiyonları düzeyinde) sabitlenmiştir. Deneyler aşağıdaki donanım konfigürasyonunda referans alınarak koşturulmuştur:
  - **Kullanılan cihaz**: `cuda`
  - **GPU**: NVIDIA RTX 4060
  - **GPU Belleği**: 8 GB
  - **RAM**: 16 GB

## Deneysel Sonuçlar (Experimental Results)

MVTec AD veri setindeki tüm 15 kategori üzerinde yapılan karşılaştırmalı deney sonuçlarının ortalamaları aşağıda özetlenmiştir. Hibrit yaklaşım için CLIP eşik değeri (threshold quantile) **0.93** olarak seçilmiştir:

| Yöntem | İmaj AUROC | Piksel AUROC (E2E) | Piksel AUROC (Cond) | Flag Rate | Toplam | Per Image | FPS |
|---|---|---|---|---|---|---|---|
| **CLIP-only** | **82.58%** | - | - | - | 113.1s | 66ms | 15.3 |
| **DINO-Max** | **85.55%** | **95.61%** | - | 100% | 812.7s | 471ms | 2.1 |
| **Hybrid** | 82.58% | 81.31% | **95.22%** ⭐ | **50.5%** | **532.6s** ✅ | 309ms | 3.2 |

**Not:** 1725 test görüntüsü üzerinden ölçüldü (15 kategori toplam). Hybrid yöntemi DINO-Max'e göre **%34.5 daha hızlı**.

### Sonuç Analizi

**✅ Ana Bulgular:**

1. **Hız Kazancı:** Hybrid yöntemi, DINO-Max'e göre **%34.5 daha hızlı** (532.6s vs 812.7s)
   - Bu, ortalama **%50.5 flag rate** sayesinde elde edilmiştir
   - Sadece şüpheli görüntüler DINOv2'ye gönderilmiştir

2. **Lokalizasyon Kalitesi:** DINO çalıştığında (flagged images) lokalizasyon kalitesi **neredeyse aynı**
   - Conditional Pixel AUROC: **95.22%** (Hybrid) vs 95.61% (DINO-Max)
   - Fark sadece **0.39 puan** - istatistiksel olarak ihmal edilebilir

3. **Trade-off:** End-to-End Pixel AUROC düşük (81.31%) çünkü:
   - CLIP bazı anomalileri kaçırıyor (false negatives)
   - Bu görüntüler DINOv2'ye gönderilmediği için lokalizasyon yapılamıyor
   - Ancak bu, **hız kazancı** için kabul edilebilir bir trade-off

4. **Image-Level Performans:**
   - CLIP ve Hybrid aynı (82.58%) - çünkü Hybrid, CLIP skorlarını kullanıyor
   - DINO-Max biraz daha iyi (85.55%) ama fark küçük (+2.97 puan)

### Kategorilere Göre Performans

**En İyi Kategoriler (Hybrid başarılı):**
- Bottle: Image 99.0%, Pixel 98.3%, Flag 73.5%
- Leather: Image 99.5%, Pixel 98.9%, Flag 79.0%
- Wood: Image 99.8%, Pixel 94.9%, Flag 77.2%
- Tile: Image 99.0%, Pixel 94.7%, Flag 76.9%

**Zorlu Kategoriler (CLIP zayıf):**
- Cable: Image 58.9% (DINO-Max: 87.5%), Flag sadece %32
- Capsule: Image 67.9% (DINO-Max: 77.4%), Flag sadece %24
- Screw: Image 53.7% (DINO-Max: 64.2%), Flag sadece %19

**Sonuç:** Hybrid yöntemi, CLIP'in güçlü olduğu kategorilerde mükemmel çalışıyor. Zorlu kategorilerde (cable, capsule) DINO-Max tercih edilebilir.

### Eşik Değeri Seçimi (Ablation Study)
Hybrid yapının tam olarak hangi CLIP Skoruna (Quantile) göre fotoğrafları DINOv2'ye yönlendireceğini belirlemek için sistemde bir "Ablation Taraması" (%90, %93, %95, %97, %99 quantile) gerçekleştirilmiştir. Aşağıdaki tabloda optimal dengenin (`0.93 - %93` Quantile) nasıl seçildiği açıkça görülmektedir:

| Seçilen Quantile (Eşik Oranı) | Ortalama İmaj AUROC | Ortalama Kondisyonel Piksel AUROC | Ortalama E2E Piksel AUROC | DINO'ya Yönlendirme Oranı (Flag) |
|---|---|---|---|---|
| 0.90 | 0.8258 | 0.9526 | **0.8273** | %55.2 |
| **0.93 (Seçilen)** | **0.8258** | **0.9522** | 0.8131 | **%50.47** ⭐ |
| 0.95 | 0.8258 | 0.9512 | 0.8037 | %47.7 |
| 0.97 | 0.8258 | 0.9541 | 0.7879 | %44.7 |
| 0.99 | 0.8258 | 0.9538 | 0.7674 | %39.6 |

**Neden 0.93 seçildi?**
- ✅ Flag rate %50.47 → Orta seviye tasarruf
- ✅ Conditional Pixel AUROC %95.22 → Mükemmel lokalizasyon kalitesi
- ✅ E2E Pixel AUROC %81.31 → Makul trade-off
- ✅ Denge noktası: Hız ve kalite arasında optimal

## Görsel Analiz (Qualitative Results)

Hibrit anomali tespit pipeline'ının nasıl çalıştığını göstermek adına üretilen 3-panelli kalitatif görseller aşağıdadır. 
**[Panel 1]** Görüntü ve CLIP Eşik Durumu | **[Panel 2]** Ground Truth Hata Maskesi | **[Panel 3]** DINOv2 Isı Haritası (Heatmap) Lokalizasyonu

![Bottle Anomaly Example](results/qual_panels_balanced/bottle/bottle_00_flagT_clip0.028.png)
![Capsule Anomaly Example](results/qual_panels_balanced/capsule/capsule_00_flagT_clip0.059.png)
![Carpet Anomaly Example](results/qual_panels_balanced/carpet/carpet_00_flagT_clip0.046.png)
![Grid Anomaly Example](results/qual_panels_balanced/grid/grid_00_flagT_clip0.035.png)

*(Daha fazla görsel örnek ve detay `results/qual_panels_balanced/` dizini altında bulunabilir.)*

## Çıktılar ve Analiz

Sistem üzerinden aşağıdaki analiz ve değerlendirme hedeflerine ulaşılır:
- Her kategori ve denenen yöntem için ayrı ayrı **AUROC** metriklerinin (görüntü bazlı ve piksel bazlı) hesaplanması.
- Karşılaştırmalı ve detaylı metriklerin **Excel** ve **CSV** formatlarında raporlanması.
- Makale için uygun olan 3-panelli kalitatif görsellerin üretilmesi:
  - Panel 1: Orijinal görüntü, CLIP skoru ve eşik durumu.
  - Panel 2: Ground-truth maske (MVTec AD referansları).
  - Panel 3: DINOv2 ısı haritası (lokalizasyon sonucu).

## Karmaşıklık Analizi (Computational Complexity)

### Teorik Big-O Analizi

**Parametreler:**
- N: Test görüntü sayısı
- P: Görüntü başına patch sayısı (~256-1024)
- B: Patch bank boyutu (~12,000)
- k: k-NN komşu sayısı (k=5)
- α: Flag rate (0 ≤ α ≤ 1)

**Yöntemler:**

1. **CLIP-only:** O(N)
   - Her görüntü için sabit işlem
   
2. **DINO-Max:** O(N · P · log(B))
   - Her görüntü için P patch × k-NN arama
   
3. **Hybrid:** O(N) + O(N · α · P · log(B))
   - CLIP gating: O(N)
   - DINOv2 (sadece flagged): O(N · α · P · log(B))
   - α ≈ 0.50 → toplam çalışma süresinde **~%34.5 tasarruf**

### Empirical Runtime

| Yöntem | Per Kategori | Toplam (15 kategori) | Per Image | FPS | Tasarruf |
|---|---|---|---|---|---|
| CLIP-only | 7.5s | 113.1s | 66ms | 15.3 | - |
| DINO-Max | 54.2s | 812.7s | 471ms | 2.1 | - |
| **Hybrid** | **35.5s** | **532.6s** | **309ms** | **3.2** | **-34.5%** ⚡ |

## Kullanım

Başlamak için depoda yer alan `Hybrid-ZeroShot-Anomaly-Detection.ipynb` Google Colab ortamında veya Jupyter çevresinde çalıştırılabilir. 

**Otomatik Ortam Tespiti:** Notebook hem local hem de Colab ortamında çalışır:
- Colab'da: Otomatik Drive mount + Colab path'leri
- Local'de: Windows/Linux/Mac path'leri

Çalıştırmadan önce `MVTEC_ROOT` dizin yolunu kendi sisteminize uygun şekilde düzenlemeniz tavsiye edilir.

## Lisans ve Referanslar

Bu proje akademik araştırma amacıyla hazırlanmıştır. Kullanılan modeller:
- **CLIP:** OpenAI (https://github.com/openai/CLIP)
- **DINOv2:** Meta AI (https://github.com/facebookresearch/dinov2)
- **MVTec AD:** MVTec Software GmbH (https://www.mvtec.com/company/research/datasets/mvtec-ad)