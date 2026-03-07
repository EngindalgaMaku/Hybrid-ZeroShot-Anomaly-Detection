# Comparison.xlsx Dosyası Açıklaması

## 📊 Dosya Yapısı

Bu Excel dosyası üç farklı yöntemin (CLIP-only, DINO-only, Hybrid) performansını karşılaştırır.

---

## 📋 Sheet'ler

### 1. **image_auroc** - Görüntü Düzeyinde AUROC

| Yöntem | Açıklama | Image AUROC Nasıl Hesaplanır |
|--------|----------|------------------------------|
| **CLIP-only** | Sadece CLIP image embeddings | `1.0 - cosine_similarity(test_img, ref_img)` |
| **DINO-only (MAX)** | Patch anomaly map → MAX aggregation | `max(anomaly_map)` ← Literatür standardı |
| **Hybrid** | CLIP skorlarını kullanır | `1.0 - cosine_similarity(test_img, ref_img)` |

**⚠️ ÖNEMLİ:** 
- **CLIP ve Hybrid'in Image AUROC değerleri AYNI çünkü Hybrid yöntemi image-level skorlama için CLIP skorlarını kullanır!**
- Hybrid'in avantajı pixel-level lokalizasyonda ortaya çıkar, image-level'da değil.

**Sonuçlar:**
```
Ortalama Image AUROC:
- CLIP-only:  82.58%
- DINO-only:  85.55%  ← En yüksek
- Hybrid:     82.58%  (CLIP ile aynı)
```

---

### 2. **pixel_e2e_auroc** - End-to-End Pixel AUROC

| Yöntem | Açıklama | Pixel E2E Nedir? |
|--------|----------|------------------|
| **DINO-only** | Tüm görüntüler için lokalizasyon | Her görüntüye DINO çalıştırılır |
| **Hybrid** | Sadece flagged görüntüler için lokalizasyon | CLIP > threshold → DINO çalışır |

**End-to-End (E2E) Ne Demek?**
- Tüm test görüntülerinin pixel-level AUROC'u
- CLIP'in kaçırdığı anomaliler (false negatives) dahil
- Hybrid için: CLIP false negative → DINO çalışmadı → sıfır anomaly map → Düşük pixel AUROC

**Sonuçlar:**
```
Ortalama Pixel E2E AUROC:
- DINO-only:  95.61%  ← En yüksek (her görüntüye DINO)
- Hybrid:     81.31%  (CLIP false negatives nedeniyle düşük)
```

**Neden Hybrid Daha Düşük?**
- CLIP bazı anomalileri kaçırır (örn: cable %58.9 image AUROC)
- Bu görüntüler DINOv2'ye gönderilmez
- Pixel-level lokalizasyon yapılamaz → E2E AUROC düşer
- **Trade-off:** Hız kazancı için kabul edilebilir

---

### 3. **pixel_cond_auroc** - Conditional Pixel AUROC

| Yöntem | Açıklama | Conditional Ne Demek? |
|--------|----------|-----------------------|
| **Hybrid** | Sadece DINO çalışan görüntülerde pixel AUROC | CLIP flagged → DINO çalıştı |

**Conditional (Koşullu) Ne Demek?**
- Sadece CLIP'in "şüpheli" dediği görüntülerde pixel AUROC
- DINO'nun gerçek lokalizasyon kalitesini gösterir
- CLIP false negatives dahil değil

**Sonuçlar:**
```
Ortalama Pixel Conditional AUROC:
- Hybrid:  95.22%  ← DINO ile neredeyse aynı! (95.61%)
```

**Ana Bulgu:**
- **DINO çalıştığında lokalizasyon kalitesi neredeyse aynı!**
- Fark sadece 0.39 puan (95.22% vs 95.61%)
- **Hybrid'in avantajı:** %50.5 flag rate → %36 daha hızlı

---

## 🎯 ÖZET: NEDEN AYNI DEĞERLER VAR?

### Image AUROC: CLIP = Hybrid ✅ NORMAL!

**Sebep:** Hybrid yöntemi image-level skorlama için CLIP kullanır.

```python
# Hybrid pipeline:
clip_score = clip_anomaly_score(img, ref)  # Image-level
if clip_score > threshold:
    dino_map = dino_localization(img)      # Pixel-level (conditional)
else:
    dino_map = zeros                       # DINO çalışmadı
```

### Pixel E2E: DINO > Hybrid ✅ NORMAL!

**Sebep:** CLIP false negatives → DINO çalışmadı → Düşük E2E pixel AUROC

### Pixel Cond: DINO ≈ Hybrid ✅ BEKLENİYOR!

**Sebep:** DINO çalıştığında lokalizasyon kalitesi aynı

---

## 📈 PERFORMANS KARŞILAŞTIRMASI

### Kategori Bazında Örnekler:

#### ✅ **Bottle** (CLIP güçlü):
```
Method      | Image AUROC | Pixel E2E | Pixel Cond | Flag Rate
------------|-------------|-----------|------------|----------
CLIP-only   | 99.0%       | -         | -          | -
DINO-only   | 94.6%       | 98.3%     | -          | 100%
Hybrid      | 99.0%       | 97.2%     | 98.3%      | 73.5%
```
- CLIP çok iyi → Hybrid mükemmel çalışıyor
- Pixel Cond neredeyse DINO ile aynı (98.3%)

#### ❌ **Cable** (CLIP zayıf):
```
Method      | Image AUROC | Pixel E2E | Pixel Cond | Flag Rate
------------|-------------|-----------|------------|----------
CLIP-only   | 58.9%       | -         | -          | -
DINO-only   | 87.5%       | 94.9%     | -          | 100%
Hybrid      | 58.9%       | 69.7%     | 96.8%      | 32.0%
```
- CLIP zayıf → Çok anomali kaçırıyor
- Pixel E2E düşük (69.7%) ama Pixel Cond yüksek (96.8%)!
- **Yorum:** DINO çalıştığında hala çok iyi lokalizasyon

---

## 🚀 ANA BULGULAR

1. **Image-Level:** CLIP ve Hybrid aynı (82.58%) - CLIP skorlarını kullanıyor ✅
2. **Pixel E2E:** Hybrid düşük (81.31%) - CLIP false negatives ❌
3. **Pixel Conditional:** Hybrid yüksek (95.22%) - DINO kalitesi korunuyor ✅
4. **Hız:** Hybrid %36 daha hızlı (503.8s vs 786.1s) ⚡

**Trade-off:**
- ✅ Hız: %36 kazanç
- ✅ Lokalizasyon (DINO çalışınca): Neredeyse aynı kalite
- ❌ E2E Pixel AUROC: Düşük (CLIP false negatives)

**Sonuç:** Hybrid, CLIP'in güçlü olduğu kategorilerde mükemmel çalışır. Zorlu kategorilerde (cable, capsule, screw) DINO-only tercih edilebilir.

---

## 📊 KULLANIM

### Excel'i Açma:
```python
import pandas as pd

# Image AUROC
df_img = pd.read_excel('comparison.xlsx', sheet_name='image_auroc')

# Pixel E2E AUROC
df_pix = pd.read_excel('comparison.xlsx', sheet_name='pixel_e2e_auroc')

# Pixel Conditional AUROC
df_cond = pd.read_excel('comparison.xlsx', sheet_name='pixel_cond_auroc')
```

### Raw Data:
```python
# Ham veriler
df_raw = pd.read_excel('comparison.xlsx', sheet_name='raw')
```

---

## ❓ SSS

**S: Neden CLIP ve Hybrid'in image AUROC'ları aynı?**
A: Hybrid, image-level skorlama için CLIP kullanır. Farkı pixel-level'da görürsün.

**S: Hybrid'in pixel E2E AUROC'u neden düşük?**
A: CLIP bazı anomalileri kaçırır (false negatives). Bu görüntülerde DINO çalışmaz → Lokalizasyon yok → Düşük E2E AUROC.

**S: Hybrid'in avantajı nedir?**
A: Hız! %50.5 flag rate ile %36 daha hızlı. DINO çalıştığında lokalizasyon kalitesi neredeyse aynı (95.22% vs 95.61%).

**S: Hangi yöntemi seçmeliyim?**
A: 
- CLIP güçlü kategoriler (bottle, leather, wood) → **Hybrid**
- CLIP zayıf kategoriler (cable, capsule, screw) → **DINO-only**
- Genel kullanım → **Hybrid** (iyi denge)

---

## 📅 Oluşturulma Tarihi

2026-03-07

## 📝 Not

Bu dosya, comparison.xlsx'teki metriklerin ne anlama geldiğini ve neden bazı değerlerin aynı olduğunu açıklar.