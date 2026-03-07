6. Nicel Sonuçlar

6.1 Ortalama Performans ve Çalışma Süresi Karşılaştırması

Önerilen hibrit anomali tespit yaklaşımının performansını değerlendirmek amacıyla üç farklı yöntem karşılaştırılmıştır: CLIP-only, DINO-max ve önerilen Hybrid yöntem. CLIP-only yaklaşımında görüntü düzeyindeki anomali tespiti yalnızca CLIP temsilleri kullanılarak gerçekleştirilmektedir. DINO-max yönteminde ise DINOv2 tabanlı anomali haritası üretilmekte ve görüntü düzeyindeki skor bu haritadaki maksimum değer üzerinden elde edilmektedir. Önerilen Hybrid yöntemde ise CLIP modeli görüntü düzeyinde bir ön tarama mekanizması olarak kullanılmakta, yalnızca eşik değerini aşan görüntüler için DINOv2 tabanlı lokalizasyon uygulanmaktadır. Hibrit yaklaşım için CLIP eşik değeri 0.93 olarak seçilmiştir.

Bu üç yöntemin ortalama performans ve çalışma süresi değerleri Tablo 2’de verilmektedir.

 
Tablo 2. Ortalama performans ve çalışma süresi karşılaştırması
Yöntem	İmaj AUROC	Piksel AUROC
(E2E)	Piksel AUROC
(Cond)	Flag Rate	Toplam	Görüntü Başına Süre	FPS
CLIP-only	82.58%	–	–	–	176.7 s	102 ms	9.8
DINO-max	85.55%	95.61%	–	100.0%	1171.5 s	679 ms	1.5
Önerilen Yöntem (Hybrid)	82.58%	81.31%	95.22%	50.5%	804.2 s	466 ms	2.1

Not: Süre ölçümleri toplam 1725 test görüntüsü üzerinden elde edilmiştir. Hybrid yöntemi, DINO-max yaklaşımına göre toplam çalışma süresinde yaklaşık %31.4 azalma sağlamıştır.

Tablo 2 incelendiğinde, CLIP-only yaklaşımının en düşük işlem süresine sahip olduğu görülmektedir. Görüntü başına süre 102 ms, işlem hızı ise 9.8 FPS olarak ölçülmüştür. Bununla birlikte bu yöntem yalnızca görüntü düzeyinde anomali tespiti gerçekleştirdiği için piksel düzeyinde lokalizasyon üretmemektedir.
DINO-max yöntemi, görüntü düzeyinde 85.55% Image AUROC ve piksel düzeyinde 95.61% Pixel AUROC (E2E) ile en yüksek genel tespit ve lokalizasyon performansını sağlamaktadır. Ancak bu yöntem aynı zamanda en yüksek hesaplama maliyetine sahiptir. Görüntü başına süre 679 ms, işlem hızı ise 1.5 FPS olarak ölçülmüştür.

Önerilen Hybrid yöntem, görüntü düzeyinde 82.58% Image AUROC, uçtan uca değerlendirmede 81.31% Pixel AUROC (E2E) ve yalnızca ikinci aşamada DINOv2 analizine yönlendirilen görüntüler üzerinde 95.22% Pixel AUROC (Cond.) elde etmiştir. Ortalama 50.5% flag rate değeri, test görüntülerinin yaklaşık yarısının ikinci aşamadaki DINOv2 analizine yönlendirildiğini göstermektedir. Çalışma süresi açısından Hybrid yöntem, 466 ms görüntü başına süre ve 2.1 FPS değeri ile DINO-max yaklaşımına kıyasla daha verimli bir yapı sunmaktadır.

Bu sonuçlar birlikte değerlendirildiğinde, önerilen Hybrid yaklaşımın DINOv2 tabanlı lokalizasyon başarısını koşullu değerlendirmede büyük ölçüde koruduğu görülmektedir. Aynı zamanda ayrıntılı patch tabanlı analizleri yalnızca gerekli durumlarda çalıştırdığı için işlem süresi bakımından DINO-max yöntemine göre daha düşük maliyet üretmektedir. Bu yönüyle hibrit sistem, doğruluk ile hesaplama verimliliği arasında daha dengeli bir çözüm sunmaktadır.

6.2 Performans – Hesaplama Maliyeti İlişkisi

Önerilen hibrit sistemde kullanılan eşik değeri, CLIP tarafından üretilen anomali skorlarının quantile tabanlı analizi ile belirlenmektedir. Farklı quantile değerlerinin sistem performansı ve hesaplama maliyeti üzerindeki etkisini incelemek amacıyla bir eşik ablation analizi gerçekleştirilmiştir. Bu analizde farklı quantile değerleri için sistemin piksel düzeyindeki performansı ve ikinci aşamada DINOv2 modelinin çalıştırılma oranı ölçülmüştür.

Farklı quantile değerleri için elde edilen sonuçlar Tablo 3’te sunulmaktadır.

Tablo 3. Quantile tabanlı performans ve DINO çalışma oranı
Quantile	Pixel AUROC (E2E)	  Pixel AUROC (Cond.)	Flag Rate
0.90	0.8273	0.9526	0.5520
0.93	0.8131	0.9522	0.5047
0.95	0.8037	0.9512	0.4766
0.97	0.7879	0.9541	0.4469
0.99	0.7674	0.9538	0.3963

Tablo 3 incelendiğinde, quantile değeri küçüldükçe DINOv2 modelinin daha fazla görüntü için çalıştırıldığı görülmektedir. Örneğin q=0.90 değerinde DINOv2 modeli test görüntülerinin yaklaşık %55.2’sinde çalıştırılırken, q=0.99 değerinde bu oran %39.6’ya düşmektedir. DINOv2’nin daha fazla görüntüde çalıştırılması, uçtan uca piksel düzeyindeki performansı artırmakta; ancak aynı zamanda işlem yükünü de yükseltmektedir. Buna karşılık quantile değeri büyüdükçe ikinci aşamaya yönlendirilen görüntü sayısı azalmakta ve hesaplama maliyeti düşmektedir.

Şekil 2’de quantile değerine bağlı olarak sistemin piksel düzeyindeki performansı ile DINOv2 çalışma oranındaki değişim gösterilmektedir.

 
Şekil 2. Quantile eşik değerine bağlı olarak hibrit sistemin performans ve hesaplama davranışı.

Sonuçlar birlikte değerlendirildiğinde, hibrit sistemin performans ile hesaplama maliyeti arasında ayarlanabilir bir denge sunduğu görülmektedir. Düşük quantile değerleri daha yüksek uçtan uca piksel performansı sağlamakta, ancak DINOv2’nin daha fazla görüntüde çalıştırılması nedeniyle işlem maliyetini artırmaktadır. Daha yüksek quantile değerleri ise hesaplama yükünü azaltmakta, buna karşılık uçtan uca piksel performansında düşüşe yol açmaktadır.

Deney sonuçlarına göre q=0.93 değeri, bu iki uç durum arasında dengeli bir çalışma noktası sunmaktadır. Bu eşik değerinde DINOv2 modeli test görüntülerinin yaklaşık %50,5’inde çalıştırılmaktadır. Böylece tüm görüntüler için patch tabanlı analiz gerçekleştiren bir DINO-max yaklaşımına kıyasla önemli ölçüde hesaplama tasarrufu sağlanırken, koşullu lokalizasyon başarısı büyük ölçüde korunmaktadır.

6.3 Kategori Bazlı Image-Level Sonuçlar

Önerilen yöntemlerin farklı ürün kategorilerindeki davranışını daha ayrıntılı incelemek amacıyla MVTec AD veri setinde yer alan 15 kategori için görüntü düzeyinde elde edilen AUROC değerleri analiz edilmiştir. Kategori bazlı sonuçlar Tablo 4’te sunulmaktadır.

Tablo 4. MVTec-AD veri setinde kategori bazlı Image AUROC karşılaştırması
Kategori	CLIP-only	DINO-max	Hybrid	Kategori	CLIP-only	DINO-max	Hybrid
bottle	0.9913	0.9460	0.9913	pill	0.7316	0.8161	0.7316
cable	0.5892	0.8746	0.5892	screw	0.5366	0.6423	0.5366
capsule	0.6769	0.7742	0.6769	tile	0.9899	0.8824	0.9899
carpet	0.9743	0.8050	0.9743	toothbrush	0.8667	0.8889	0.8667
grid	0.7193	0.9883	0.7193	transistor	0.7925	0.7963	0.7925
hazelnut	0.8614	0.8746	0.8614	wood	0.9982	0.8281	0.9982
leather	0.9949	0.9392	0.9949	zipper	0.8997	0.8529	0.8997
metal_nut	0.7771	0.9228	0.7771				
Ortalama	0.8266	0.8554	0.8266	Std. Sapma	0.1532	0.0847	0.1532

Not: Hybrid yöntemde görüntü düzeyindeki anomali skoru doğrudan CLIP modeli tarafından üretildiği için, kategori bazlı Image AUROC değerleri CLIP-only yöntemi ile aynıdır.

Tablo 4 incelendiğinde, CLIP-only ve DINO-max yöntemlerinin farklı kategori türlerinde farklı güçlü yönlere sahip olduğu görülmektedir. CLIP-only yaklaşımı özellikle bottle, carpet, leather, tile ve wood gibi kategorilerde yüksek görüntü düzeyinde ayrım başarısı elde etmektedir. Bu kategorilerde anomaliler çoğunlukla ürünün genel görsel yapısını etkileyen değişimler içerdiğinden, küresel temsil öğrenimi yapan CLIP modeli başarılı sonuçlar üretmektedir.

Buna karşılık cable, capsule, grid, metal_nut, pill ve screw gibi kategorilerde DINO-max yöntemi daha yüksek performans göstermektedir. Bu kategorilerde anomaliler genellikle küçük yapısal bozulmalar veya yerel kusurlar şeklinde ortaya çıktığından, patch tabanlı temsil kullanan DINOv2 temelli yaklaşım daha etkili sonuçlar üretmektedir.

Kategori bazlı sonuçlar, Hybrid yaklaşımın CLIP’in güçlü olduğu kategorilerde yüksek başarıyı koruduğunu göstermektedir. Özellikle bottle, leather, wood ve tile kategorilerinde görüntü düzeyindeki başarı yüksek kalmaktadır. Buna karşılık cable, capsule ve screw gibi kategorilerde görüntü düzeyindeki ayrım başarısı daha düşüktür. Bu kategorilerde DINO-max yönteminin daha yüksek görüntü düzeyi performans üretmesi, anomalilerin daha çok yerel yapısal bozulmalar şeklinde ortaya çıktığını düşündürmektedir.

Hybrid yöntemde görüntü düzeyindeki skor CLIP tarafından üretildiği için, Image AUROC değerleri kategori bazında CLIP-only ile aynı kalmaktadır. Bu durum, hibrit sistemin görüntü düzeyindeki karar mekanizmasını CLIP üzerinden koruduğunu göstermektedir. Buna karşılık yöntemin temel avantajı, yalnızca şüpheli görüntüler için DINOv2 tabanlı ikinci aşamayı devreye alarak hesaplama maliyetini azaltması ve gerekli durumlarda ayrıntılı lokalizasyon üretebilmesidir. Bu nedenle Hybrid yaklaşım, CLIP’in güçlü olduğu kategorilerde yüksek doğruluk ve daha düşük hesaplama maliyeti arasında etkili bir denge sunmaktadır. Zorlu kategorilerde ise DINO-max yaklaşımı daha yüksek görüntü düzeyi başarı sağlayabilmektedir.

6.4 Hesaplama Karmaşıklığı ve Çalışma Süresi Analizi

Önerilen hibrit yaklaşımın hesaplama verimliliğini incelemek amacıyla yöntemlerin hem pratik çalışma süreleri hem de ampirik ölçeklenme davranışları analiz edilmiştir. Bu analiz, CLIP-only, DINO-max ve önerilen Hybrid yöntemleri için gerçekleştirilmiştir.

Tüm test veri kümesi üzerinde ölçülen ortalama çalışma süreleri Tablo 5’te verilmektedir.

Tablo 5. Yöntemlerin ortalama çalışma süresi karşılaştırması
Yöntem	Toplam Süre (s)	Görüntü Başına Süre	FPS	Göreli Süre
CLIP-only	176.7	102 ms	9.8	1.00×
DINO-max	1171.5	679 ms	1.5	6.63×
Hybrid	804.2	466 ms	2.1	4.55×

Tablo 5 incelendiğinde, CLIP-only yönteminin yalnızca küresel özellik çıkarımı yaptığı için en düşük hesaplama maliyetine sahip olduğu görülmektedir. DINO-max yöntemi ise tüm görüntüler için patch tabanlı analiz gerçekleştirdiğinden en yüksek çalışma süresine sahiptir. Önerilen Hybrid yöntem, CLIP modelini hızlı bir ön tarama mekanizması olarak kullanarak yalnızca şüpheli görüntüler için DINOv2 tabanlı analiz gerçekleştirmektedir. Bu sayede DINO-max yöntemine kıyasla belirgin bir süre avantajı sağlamaktadır. Toplam süre açısından Hybrid yaklaşım, DINO-max yöntemine göre yaklaşık %31.4 daha düşük maliyet üretmektedir.

Yöntemlerin ölçeklenme davranışını incelemek amacıyla farklı test görüntüsü sayıları kullanılarak log-log regresyonuna dayalı bir ampirik karmaşıklık analizi de gerçekleştirilmiştir. Elde edilen eğim değerleri Tablo 6’da sunulmaktadır.

Tablo 6. Yöntemlerin karmaşıklık analizi sonuçları
Yöntem	Log-Log Eğim	Karmaşıklık Eğilimi	Ortalama Süre (s)
CLIP-only	0.915	Yaklaşık doğrusal	0.76
DINO-max	1.042	Yaklaşık doğrusal	10.92
Hybrid	1.557	Ölçülen aralıkta doğrusal üstü	6.10

Tablo 6’daki sonuçlar, CLIP-only ve DINO-max yöntemlerinin test görüntüsü sayısı ile yaklaşık doğrusal biçimde ölçeklendiğini göstermektedir. Hybrid yaklaşımı ise ölçülen deney aralığında daha yüksek bir eğim sergilemektedir. Bu durum, hibrit sistemde kullanılan eşik tabanlı yönlendirme mekanizmasının farklı veri alt kümelerinde DINOv2 analizini farklı oranlarda tetiklemesiyle ilişkilidir. Başka bir deyişle, hibrit yapının çalışma süresi yalnızca görüntü sayısına değil, ikinci aşamaya yönlendirilen örnek oranına da bağlıdır.

Bu sonuçlar birlikte değerlendirildiğinde, önerilen Hybrid yaklaşımın doğruluk ve hesaplama maliyeti arasında dengeli bir çözüm sunduğu görülmektedir. Özellikle tüm görüntüler için pahalı patch tabanlı analiz gerçekleştiren DINO-max yöntemine kıyasla, yalnızca anomali olasılığı yüksek görülen görüntüler üzerinde ayrıntılı analiz yapılması sistemin pratik uygulanabilirliğini artırmaktadır.
