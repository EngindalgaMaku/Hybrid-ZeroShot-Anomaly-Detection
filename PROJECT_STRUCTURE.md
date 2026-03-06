# 📁 Proje Yapısı

Bu belge projedeki dosyaların organizasyonunu açıklar.

## 🎯 Ana Dosyalar

### Notebook'lar
- **`Hybrid-ZeroShot-Anomaly-Detection-LOCAL.ipynb`** - Ana çalışma notebook'u (Lokal)
- **`Hybrid-ZeroShot-Anomaly-Detection.ipynb`** - Google Colab versiyonu

### Python Modülleri
- **`CLS_TOKEN_UPDATE_CELLS.py`** - DINOv2 CLS token fonksiyonları
  - `dinov2_cls_token()` - CLS token çıkarma
  - `build_dino_cls_bank()` - CLS referans bank
  - `dino_cls_image_score()` - Image-level scoring
  - `run_category_hybrid_CLS()` - Hybrid method with CLS

### Dökümanlar
- **`README.md`** - Proje ana dökümanı
- **`README_EN.md`** - English version
- **`requirements.txt`** - Python bağımlılıkları

## 📊 Sonuçlar (`experiments/`)

```
experiments/hybrid_results/
├── 01_hybrid_with_cls/          # Hybrid (CLS token ile)
├── 02_clip_only/                # CLIP-only baseline
├── 03_dino_only_cls/            # DINO-only (CLS ile)
├── comprehensive_report/        # Karşılaştırma raporu
│   ├── comprehensive_comparison.xlsx
│   ├── COMPREHENSIVE_REPORT.md
│   ├── performance_comparison.png/pdf
│   ├── timing_comparison.png/pdf
│   └── tradeoff_curve.png/pdf
└── qual_panels_balanced/        # Qualitative örnekler
```

## 📚 Dökümanlar (`docs/`)

- **`CLS_TOKEN_FINAL_REPORT.md`** - CLS token deneylerinin detaylı raporu
- **`HYBRID_PIXEL_AUROC_ANALYSIS.md`** - Pixel AUROC analizi
- **`ADAPTIVE_THRESHOLD_GUIDE.md`** - Adaptive threshold kılavuzu
- **`ADAPTIVE_THRESHOLD_README.md`** - Threshold optimization
- **`LOCAL_SETUP.md`** - Lokal kurulum talimatları

## 🗄️ Arşiv (`archive/`)

Eski deneyler ve kullanılmayan kodlar:
- `analyze_false_negatives.py` - False negative analiz script'i
- `visualize_false_negatives.py` - FN görselleştirme
- `ensemble_gating.py` - Ensemble gating implementasyonu
- `COMPREHENSIVE_COMPARISON_REPORT.py` - Rapor generator
- `complexity_analysis.py` - Kompleksite analizi
- `adaptive_threshold_optimizer.py` - Eski threshold optimizer

## 🚀 Hızlı Başlangıç

### 1. Lokal Notebook Çalıştır
```bash
jupyter notebook Hybrid-ZeroShot-Anomaly-Detection-LOCAL.ipynb
```

### 2. Temel Kullanım
```python
# CLS fonksiyonlarını yükle
exec(open('CLS_TOKEN_UPDATE_CELLS.py', encoding='utf-8').read())

# Tek kategori test et
result = run_category_hybrid_CLS('bottle')

# Tüm kategorileri çalıştır
df_dino, df_hybrid = test_all_categories_CLS(save_results=True)
```

## 📝 Notlar

- Ana geliştirme: `LOCAL.ipynb` dosyasında yapılır
- Sonuçlar otomatik olarak `experiments/` klasörüne kaydedilir
- Archive klasöründeki dosyalar eski deneylerdir, silinebilir
- Docs klasöründeki dökümanlar referans amaçlıdır