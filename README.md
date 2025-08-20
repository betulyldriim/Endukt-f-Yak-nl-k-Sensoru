1. Genel Bakış
Proje, bir pompanın veya benzeri bir dönen makinenin hızını ölçmek, bu verileri işlemek ve bir HMI (İnsan-Makine Arayüzü) paneli üzerinden görselleştirmek için tasarlanmıştır. Sistem, hız ölçümü için bir yakınlık sensörü kullanmakta, bu sensörden gelen darbe sinyalleri PLC tarafından işlenmekte ve sonuçlar HMI ekranına aktarılmaktadır.

2. Kullanılan Donanım ve Yazılım
PLC: PLC_1 [CPU 1214C DC/DC/DC]. Bu, Siemens'in SIMATIC S7-1200 serisinden, DC güç kaynağı, DC girişler ve DC çıkışlara sahip bir kompakt CPU'dur.

HMI: HMI_1 [TP700 Comfort]. Bu, Siemens'in Comfort Panel serisinden, 7 inçlik bir dokunmatik ekrana sahip bir operatör panelidir.

Yazılım: Siemens TIA Portal V16. Proje, hem PLC programlama hem de HMI tasarımı için TIA Portal'ın tek bir entegre platformunu kullanmaktadır.

Sensör: Proje açıklamasında belirtildiği gibi, hız ölçümü için bir "Proximity Sensor" (yakınlık sensörü) kullanılmıştır. Bu sensör, her bir devirde bir darbe üretecek şekilde makine miline monte edilmiştir.

3. PLC Programı Analizi (Main [OB1])
PLC programı, Ladder Logic (LAD) dilinde yazılmıştır ve ana işlevleri yerine getirmektedir.

a) Ağ 1: Hız Darbelerinin Sayılması ve Zamanlama

Blok: CTU (Up Counter - Yukarı Sayıcı)

Fonksiyon: Yakınlık sensöründen gelen darbeleri sayar.

Girişler:

CU (Count Up): "Speed_Detected" etiketi (%I0.1). Sensörden gelen darbe sinyalini alır. Her darbe geldiğinde sayıcı bir artar.

R (Reset): Sayıcıyı sıfırlamak için kullanılır. Bu resimde bir şarta bağlı değil, bu da sayıcının sürekli arttığı anlamına gelebilir.

PV (Preset Value): Ön ayar değeri. Bu sayıcıda 9999 olarak ayarlanmış, ancak bu değerin belirli bir amacı net değil, zira sayıcının değeri genellikle sürekli artar.

Çıkış: CV (Current Value): Sayıcının anlık değerini MD10 bellek adresine "Actual_Counting" etiketiyle kaydeder.

Blok: TON (On-Delay Timer - Gecikmeli Açma Zamanlayıcısı)

Fonksiyon: Belirli bir süre sonunda bir çıkışı aktif eder.

Girişler:

IN (Input): "AlwaysTRUE" (%M1.2). Bu sürekli aktif olan bir bit ile zamanlayıcının her zaman çalışmasını sağlar.

PT (Preset Time): T#180ms. Zamanlayıcının sayacağı süreyi belirtir. 180 milisaniye olarak ayarlanmıştır.

Çıkış: ET (Elapsed Time): Geçen süreyi gösterir.

Açıklama: Bu zamanlayıcı, "Bölçek 4" adında bir bit (%M0.6) her 180ms'de bir kısa süreliğine aktif etmektedir. Bu bitin diğer resimlerdeki programda ne için kullanıldığı net olmamakla birlikte, muhtemelen hız hesaplamalarının belirli periyotlarda güncellenmesi için bir tetikleyici görevi görür.

b) Ağ 2: Hız Değerlerinin Hesaplanması

Blok: MOVE

Fonksiyon: Bir değeri bir adresten başka bir adrese taşır.

Giriş: "Actual_Counting" (MD10). Sayıcının anlık değerini alır.

Çıkış: "RPS" (%MD10). "Actual_Counting" değeri doğrudan "RPS" (Revolutions Per Second - Saniyedeki Devir) olarak etiketlenmiş %MD10 adresine taşınır. Bu, sensörün bir saniyede kaç darbe ürettiğini saydığı anlamına gelir. (Not: Sayıcı 180ms'de bir sayıldığı için, doğru bir RPS değeri için "Actual_Counting" değeri 5.555 ile çarpılmalıdır, ancak bu resimde görülmüyor. Belki de bu mantık başka bir yerde uygulanmıştır).

Blok: MUL (Multiply - Çarpma)

Fonksiyon: İki değeri çarpar.

1. Çarpma İşlemi: "RPS" (IN1) değeri 60 ile (IN2) çarpılarak "RPM" (Revolutions Per Minute - Dakikadaki Devir) değeri "MD14" adresine kaydedilir.

2. Çarpma İşlemi: "RPM" (IN1) değeri tekrar 60 ile (IN2) çarpılarak "RPH" (Revolutions Per Hour - Saatteki Devir) değeri "MD18" adresine kaydedilir.

c) Ağ 3: Başlatma/Durdurma Kontrolü

Mantık: Bir mandallama (latch) devresi kullanılmıştır.

Girişler:

Start (%M0.0): HMI'daki "Start" butonuna karşılık gelir.

Stop (%M0.1): HMI'daki "Stop" butonuna karşılık gelir.

Çıkış: Pump (%Q0.1). Pompayı çalıştıran veya durduran dijital çıkıştır.

Çalışma Prensibi:

Start butonu aktif olduğunda (%M0.0), Pump (%Q0.1) çıkışı aktif olur.

Pump çıkışı kendi kontaktı üzerinden devreyi kilitler (%Q0.1 kontağı), böylece Start butonu bırakılsa bile pompa çalışmaya devam eder.

Stop butonu (%M0.1) aktif olduğunda, normalde kapalı kontağı açılarak pompa çıkışını pasif hale getirir ve mandalı kırar.

4. HMI Ekran Analizi
Başlık: "Speed Measurement using Proximity Sensor" ve "PUMP SPEED MONITORING AND CONTROL". Projenin amacını net bir şekilde belirtir.

Veri Alanları: HMI ekranında üç farklı hız değeri görüntülenmektedir:

RPS: Saniyedeki devir (Revolutions Per Second).

RPM: Dakikadaki devir (Revolutions Per Minute).

RPH: Saatteki devir (Revolutions Per Hour).

Bu veriler, PLC'deki hesaplama sonuçları olan %MD10, %MD14 ve %MD18 etiketlerine bağlanmıştır.

Kontrol Butonları:

Start: PLC'deki %M0.0 girişini tetikleyerek pompanın çalışmasını başlatır.

Stop: PLC'deki %M0.1 girişini tetikleyerek pompanın çalışmasını durdurur.

Görselleştirme: Ekranın ortasında bir motor ve pompa sembolü bulunmaktadır. Bu, sistemin durumunu görsel olarak temsil etmeye yardımcı olur.

Diğer Bilgiler: Ekranın sağ üst köşesinde güncel tarih ve saat (12/31/2000, 10:59:39 AM) ve sağ alt köşede "TOUCH" kelimesi bulunmaktadır.
