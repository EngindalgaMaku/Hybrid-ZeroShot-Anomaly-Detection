5.1 Veri Seti

Deneyler, endüstriyel anomali tespiti çalışmalarında yaygın olarak kullanılan MVTec-AD veri seti (Bergmann vd., 2021) üzerinde gerçekleştirilmiştir. Bu veri seti, farklı endüstriyel ürün türlerine ait görüntülerden oluşmakta ve hem normal hem de kusurlu örnekler içermektedir. MVTec-AD toplam 15 farklı kategori içermekte olup bu kategoriler nesne ve doku tabanlı ürünlerden oluşmaktadır.

Her kategori için veri setinde normal ürün görüntülerinin yanı sıra farklı kusur türlerini içeren anomali görüntüleri bulunmaktadır. Ayrıca veri seti, anomalilerin görüntü üzerindeki konumunu gösteren piksel seviyesinde yer doğruluk (ground truth) maskeleri de sağlamaktadır. Bu özellik sayesinde hem görüntü seviyesinde anomali tespiti hem de piksel seviyesinde lokalizasyon performansı değerlendirilebilmektedir.
Bu çalışmada deneyler, MVTec-AD veri setinde yer alan 15 kategorinin tamamı kullanılarak gerçekleştirilmiştir. 

5.2 Deney Ortamı

Deneylerin tekrarlanabilirliğini sağlamak amacıyla rastgelelik kontrol edilmiş ve tüm çalıştırmalar için random seed değeri 42 olarak sabitlenmiştir.

Deneyler CUDA destekli GPU ortamına sahip bir kişisel bilgisayar üzerinde gerçekleştirilmiştir. Kullanılan donanım bileşenleri NVIDIA GeForce RTX 4060 GPU (8 GB GPU belleği) ve 16 GB sistem belleğinden oluşmaktadır.

Tüm deneyler Python 3.14.3 (64-bit) ortamında PyTorch derin öğrenme kütüphanesi kullanılarak gerçekleştirilmiştir. Tüm yöntemler aynı donanım ortamında çalıştırılmış ve süre ölçümleri aynı sistem üzerinde yapılmıştır. Bu sayede farklı yöntemler arasındaki işlem süresi karşılaştırmaları tutarlı bir şekilde gerçekleştirilebilmiştir.

5.3 Deney Parametreleri

CLIP ve DINOv2 modellerinin birlikte kullanıldığı hibrit sistemde kullanılan temel parametreler bu bölümde sunulmaktadır. CLIP tabanlı görüntü temsilinde ViT-B/16 mimarisi kullanılmıştır. Normal görüntülerden elde edilen referans özellikler kullanılarak görüntü seviyesinde bir anomali skoru hesaplanmıştır.

DINOv2 tabanlı lokalizasyon aşamasında ise patch seviyesinde özellikler kullanılmıştır. Normal görüntülerden elde edilen patch özellikleri bir referans özellik bankası oluşturmak amacıyla saklanmış ve test görüntülerindeki patch özellikleri bu referanslar ile k-en yakın komşu (k-NN) yöntemi kullanılarak karşılaştırılmıştır.

Deneylerde CLIP tabanlı referans temsil için 50 normal görüntü, DINOv2 tabanlı patch referansları için ise 30 normal görüntü kullanılmıştır. Görüntü başına değerlendirilen maksimum patch sayısı 400 olarak belirlenmiştir. Referans özellik bankasının boyutunu kontrol etmek amacıyla 12000 patch içeren bir coreset yapısı kullanılmıştır. Patch benzerlik hesaplamalarında k = 5 komşu değeri tercih edilmiştir. Tüm deneylerde değerlendirme çözünürlüğü 256 × 256 piksel olarak sabit tutulmuştur.

Bu parametreler tüm veri seti kategorileri için aynı şekilde kullanılmıştır. Ayrıca hibrit sistemde CLIP modeli tarafından üretilen anomali skorlarının yorumlanabilmesi için bir eşik değeri kullanılmıştır. Bu eşik değeri, eğitim veri setindeki normal görüntülerin CLIP skor dağılımı üzerinden quantile tabanlı bir yöntemle belirlenmiştir. Deneylerde hibrit sistemin çalıştırılması için q = 0.93 quantile değeri kullanılmıştır.

5.4 Değerlendirme Metrikleri

Önerilen sistemin performansı hem görüntü seviyesinde hem de piksel seviyesinde değerlendirilmiştir. Görüntü seviyesinde anomali tespiti performansı AUROC (Area Under the Receiver Operating Characteristic Curve) metriği kullanılarak ölçülmüştür. Bu metrik, modelin normal ve anomali görüntülerini ayırt edebilme başarısını eşik değerinden bağımsız olarak değerlendirmektedir.

Piksel seviyesinde lokalizasyon performansı ise üretilen anomali haritaları ile veri setinde bulunan yer doğruluğu maskeleri karşılaştırılarak hesaplanan Pixel AUROC değeri kullanılarak değerlendirilmiştir.

Hibrit sistemin piksel seviyesindeki performansını daha ayrıntılı analiz edebilmek amacıyla iki farklı ölçüm gerçekleştirilmiştir. Conditional Pixel AUROC, yalnızca DINOv2 modelinin çalıştırıldığı görüntüler üzerindeki lokalizasyon başarısını ölçmektedir. End-to-End Pixel AUROC ise tüm test görüntülerini kapsayan genel piksel seviyesindeki performansı ifade etmektedir.

Bu iki metrik birlikte kullanılarak hibrit sistemin hem lokalizasyon kalitesi hem de tüm sistemin genel performansı ayrı ayrı değerlendirilebilmektedir.
