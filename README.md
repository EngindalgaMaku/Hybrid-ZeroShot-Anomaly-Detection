# Hybrid-ZeroShot-Anomaly-Detection

Bu proje, **CLIP** ve **DINOv2** modellerini kullanarak üretim/sanayi verilerinde (örn. MVTec AD veri seti) sıfır-örneklem (zero-shot) anomali tespiti yapmayı amaçlayan melez (hibrit) bir yaklaşım sunmaktadır. Bu çalışma, konuyla ilgili akademik bir makale için hazırlanmış olup kodun tam ve çalıştırılabilir halini içermektedir.

## Yöntemler ve Yaklaşımlar

Proje, üç farklı anomali tespit yaklaşımını analiz etmekte ve karşılaştırmaktadır:

1. **Hybrid (Hibrit) Yaklaşım**: 
   - **CLIP** modeli kullanılarak görüntüler ön taramadan geçirilir (image-level filtreleme). Bu aşama oldukça hızlıdır.
   - Sadece şüpheli ("flagged") olarak işaretlenen görüntüler **DINOv2** modeline gönderilerek detaylı piksel bazlı (pixel-level) lokalizasyon analizi yapılır.
   - Bu yöntem sayeside hem hız hem de yüksek detaylı lokalizasyon başarısı elde edilir.
   
2. **CLIP-only Yaklaşım**:
   - Sadece CLIP modelini kullanarak görüntü düzeyinde (image-level) anomali tespiti yapılır. Lokalizasyon sağlamaz ancak hızlı çalışır.
   
3. **DINO-only Yaklaşım**:
   - Herhangi bir ön filtreleme yapmaksızın sadece DINOv2 modeli kullanılarak piksel düzeyinde anomali tespiti ve lokalizasyon yapılır. En yüksek hesaplama maliyetine sahiptir fakat en detaylı analizi sunar.

## Kullanılan Veri Seti (MVTec AD)

Bu çalışmada, endüstriyel görsel hata tespiti (industrial visual anomaly detection) alanında standart ve yaygın olarak kullanılan halka açık bir veri seti olan **MVTec AD** (Anomaly Detection) kullanılmıştır. 
Veri setini incelemek ve indirmek için resmi bağlantıyı ziyaret edebilirsiniz:  
🔗 [MVTec AD Dataset Official Page](https://www.mvtec.com/company/research/datasets/mvtec-ad)

## Kullanılan Teknolojiler ve Tekrarlanabilirlik

- **OpenAI CLIP**: Görüntü bazlı genel anomali özelliklerini çıkarmak ve filtrelemek için.
- **Meta DINOv2**: Piksel bazlı haritalama (patch-level feature extraction), KNN indeksleme ve anomali ısı haritaları (heatmap) oluşturmak için.
- **Python Kütüphaneleri**: `pandas`, `opencv-python`, `scikit-learn`, `ftfy`, `regex`, ve veri yönetimi için PyTorch altyapısı.
- **Tekrarlanabilirlik (Reproducibility) ve Cihaz Ayarları**: Deneylerin tutarlı ve akademik anlamda tekrarlanabilir olması için rastgelelik tohumu (seed) `SEED = 42` olarak (Numpy, Random ve PyTorch CPU/GPU fonksiyonları düzeyinde) sabitlenmiştir. Deneyler aşağıdaki donanım konfigürasyonunda referans alınarak koşturulmuştur:
  - **Kullanılan cihaz**: `cuda`
  - **GPU**: NVIDIA RTX PRO 6000 Blackwell Server Edition
  - **GPU Belleği**: 101.97 GB

## Çıktılar ve Analiz

Sistem üzerinden aşağıdaki analiz ve değerlendirme hedeflerine ulaşılır:
- Her kategori ve denenen yöntem için ayrı ayrı **AUROC** metriklerinin (görüntü bazlı ve piksel bazlı) hesaplanması.
- Karşılaştırmalı ve detaylı metriklerin **Excel** ve **CSV** formatlarında raporlanması.
- Makale için uygun olan 3-panelli kalitatif görsellerin üretilmesi:
  - Panel 1: Orijinal görüntü, CLIP skoru ve eşik durumu.
  - Panel 2: Ground-truth maske (MVTec AD referansları).
  - Panel 3: DINOv2 ısı haritası (lokalizasyon sonucu).

## Kullanım

Başlamak için depoda yer alan `Hybrid-ZeroShot-Anomaly-Detection.ipynb` Google Colab ortamında veya Jupyter çevresinde çalıştırılabilir. Veri seti dizin yolları Colab içerisinde Google Drive üzerinden çekilecek şekilde ayarlanmıştır. Çalıştırmadan önce `MVTEC_ROOT` dahil dizin yapılarını kendi Google Drive veya lokal sisteminize uygun şekilde düzenlemeniz tavsiye edilir.
