# 🏛️ Muhtarlık Yönetim Sistemi Veri Tabanı Mimarisi

SQL Server (T-SQL) kullanılarak geliştirilmiş; mahalle düzeyindeki bürokratik, coğrafi, sosyal yardımlaşma, resmi evrak takibi ve vatandaş talep/şikayet süreçlerini dijitalleştiren kapsamlı, ilişkisel ve normalize edilmiş bir veri tabanı projesidir. 

Proje bünyesinde veri bütünlüğünü korumak, iş mantıklarını otomatize etmek ve sorgu performansını artırmak adına gelişmiş T-SQL programlama ögeleri (Views, Stored Procedures, Functions, Triggers) aktif olarak kullanılmıştır.

---

## 📊 Veri Tabanı İlişki Diyagramı (ERD)

Aşağıdaki şemada; vatandaş hiyerarşisi, coğrafi adres düzeni, resmi tebligatlar, sosyal yardımlar ve vatandaş talep/şikayet süreçlerinin birbiriyle olan PK-FK (Primary Key - Foreign Key) ilişkileri modellenmiştir:

![Muhtarlık Veritabanı Diyagramı](Diyagram.png)

---

## 📂 Veri Tabanı Tablo Yapısı

Sistem, bir mahallenin ve muhtarlığın tüm idari ihtiyaçlarını eksiksiz karşılayabilmesi adına normalizasyon kurallarına uygun olarak **21 adet ilişkisel tablodan** kurgulanmıştır:

* **Vatandaş ve Aile Yönetimi:** `tbl_vatandas`, `tbl_vatandasTur`, `tbl_medeniDurum`
* **Hiyerarşik Coğrafi Adres Yapısı:** `tbl_il`, `tbl_ilce`, `tbl_mahalle`, `tbl_sokak`, `tbl_bina`, `tbl_daire`, `tbl_ikamet`, `tbl_ikametgahDurumu`
* **İdari ve Bürokratik Operasyonlar:** `tbl_muhtar`, `tbl_islem`, `tbl_evrakTipii`
* **Resmi Evrak Takibi:** `tbl_tebligat`, `tbl_teslimatDurumu`
* **Sosyal Destek & Şikayet Yönetimi:** `tbl_sosyalYardim`, `tbl_sosyalYardimDurumu`, `tbl_yardimTipi`, `tbl_talebSikayet`, `tbl_talepSikayetDurumu`

---

## 🛠️ Öne Çıkan Mimari Özellikler

### 1. 👁️ Gelişmiş Sanal Tablo Mimarisi (Views)
Projede veri analizi, performans artırımı, kurumsal şifreleme ve fiziksel veri koruması sağlamak amacıyla **3 farklı yapıda** gelişmiş View tasarlanmıştır:
* **Normal View (`vw_vatandasDetayliAdres`):** Normalizasyon sebebiyle parçalanan vatandaş, ikamet ve hiyerarşik adres tablolarını `JOIN` mimarisiyle tek bir sanal yapıda birleştirir. Muhtarın karmaşık sorgular yazmadan adres bilgilerine tek bir yerden erişmesini sağlar.
* **With Encryption View (`vw_SosyalYardimGizli`):** İhtiyaç sahibi vatandaşların sosyal yardım bilgilerini barındıran bu yapının kaynak kodları, veri gizliliğini korumak amacıyla veri tabanı seviyesinde şifrelenmiştir. Yetkisiz kişilerin sorgu algoritmasını görmesi engellenir.
* **With Schemabinding View (`vw_ResmiIslemTakip`):** Muhtarlıkta yürütülen resmi evrak ve işlemlerin raporlandığı kritik yapının yanlışlıkla bozulmasını engeller. Görünümün kullandığı tabloları fiziksel olarak kilitler; view silinmeden bağlı olduğu tabloların yapısı değiştirilemez.

### 2. 📧 Dinamik E-Posta Tetikleyicisi (Trigger)
* **`trg_vatandasMailGonder`:** Muhtarlığa yeni bir vatandaş talep veya şikayet kaydı girildiği anda (`AFTER INSERT`) otomatik olarak devreye giren bir otomasyondur. Girilen veriyi algılayarak ilgili vatandaşın sistemde kayıtlı e-posta adresine anlık durum, öncelik ve sistem kayıt bilgilerini içeren şık bir HTML e-posta bildirimi (`msdb.dbo.sp_send_dbmail`) göndermektedir.

### 3. ⚡ Gelişmiş Saklı Yordamlar (Stored Procedures)
Sistem güvenliğini artırmak, veri manipülasyonunu standartlaştırmak ve ağ trafiğini azaltmak amacıyla `stp_...EkleSilGuncelle` mimarisinde saklı yordamlar geliştirilmiştir:
* **Adres ve Operasyon Yönetimi SP'leri:** `stp_ilEkleSilGuncelle`, `stp_ilceEkleSilGuncelle`, `stp_mahalleEkleSilGuncelle` gibi yapılarla dinamik işlem tipi parametresine göre güvenli CRUD süreçleri yönetilir.

### 4. 🧮 Kullanıcı Tanımlı Fonksiyonlar (Functions)
* **Tablo Döndüren Fonksiyon (`fn_CocukSayisiGetir`):** Belirtilen çocuk sayısına sahip mahalle sakinlerini aile yapısıyla birlikte listeleyen satır içi (inline) fonksiyon yapısıdır.
* **Skalar Fonksiyon (`fn_ToplamSosyalYardim`):** Belirli bir vatandaş ID'sine göre sistemde o vatandaşa yapılmış olan tüm sosyal yardımların finansal toplamını (`SUM`) hesaplayıp geriye tek bir değer döndüren optimize fonksiyondur.

---

## 🔐 Veri Bütünlüğü ve Güçlü Kısıtlamalar (Constraints)

Veri giriş kalitesini en üst düzeyde tutmak amacıyla şu kısıtlamalar veri tabanı motoru seviyesinde zorunlu kılınmıştır:
* **Check Constraints:** * T.C. Kimlik Numaraları ve telefon numaralarının tam 11 karakter uzunluğunda olması şart koşulmuştur.
  * Sokak posta kodlarının tam 5 karakter olması zorunlu tutulmuştur.
  * Sosyal yardım miktarının negatif (`< 0`) değer alması engellenmiştir.
  * E-posta alanları için `@` format kontrolü entegre edilmiştir.
* **Unique Constraints:** T.C. Kimlik No, telefon numaraları ve e-posta adreslerinin mükerrer (tekrar eden) kayıt olması engellenerek veri tutarlılığı sağlanmıştır.
* **Default Constraints:** Kayıt esnasında boş bırakılan alanlara otomatik sistem değerleri atanır (Cinsiyete `'Bilinmiyor'`, talep önceliğine `'Normal'`, sosyal yardım miktarına `0`, işlem tarihlerine ise `GETDATE()`).
