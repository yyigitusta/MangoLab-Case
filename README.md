Markdown# FX Çevirici Servisi (FX Converter Tool)

Avrupa Merkez Bankası (ECB) kurlarını frankfurter.dev üzerinden kullanarak para birimi çevirisi yapan, Python ve FastAPI ile geliştirilmiş bir HTTP servisi.

## Nasıl Çalıştırılır
```bash
./run.sh
Servis 8080 portunda (veya ortam değişkenlerinde PORT olarak belirtilen portta) ayağa kalkacaktır.

Testler Nasıl Çalıştırılır
Bash./test.sh
Testler, dış API'yi taklit etmek (mocking) için respx kütüphanesini kullanır.
Bu sayede testler internet bağlantısı olmadan da başarılı bir şekilde çalışır.
Hata Kodları (Error Codes)HTTP DurumuHata KoduAçıklama
                       400invalid_amountMiktar sıfır, negatif veya çok fazla ondalık hane içeriyor.400same_currencyKaynak ve hedef para birimleri aynı girilmiş.400invalid_date_formatTarih YYYY-MM-DD formatında değil.400future_dateİstenen tarih gelecekte bir gün.400date_too_oldİstenen tarih 1999-01-04'ten daha eski.400currency_not_foundGirilen para birimi kodu geçersiz veya bulunamadı.502upstream_timeoutDış API (Frankfurter) zaman aşımına uğradı.502upstream_errorDış API beklenmeyen bir HTTP hatası döndürdü.502upstream_unavailableDış API'ye ulaşılamıyor veya geçersiz bir JSON formatı döndü.Uç Durumların Yönetimi (Edge Case Handling)Eksik Tarihler (Hafta sonları/Tatiller): Eğer bugün için istek atıldığında dış API 404 dönüyorsa (örneğin kurlar o gün için henüz yayınlanmadıysa), sistem otomatik olarak /latest (en güncel) endpoint'ine düşer. Yanıtta kullanıcının sorduğu tarih asked_date, kurun gerçekten ait olduğu tarih ise rate_date olarak şeffafça ayrılır.Gelecek/Eski Tarihler: Dış API'ye hiç istek atılmadan, doğrudan sistem kapısında 400 hatası fırlatılarak engellenir.Geçersiz/Aynı Para Birimleri: İstek API'ye gitmeden doğrulanır. Eğer dış API para birimini bulamazsa (404), bu durum müşteriye currency_not_found olarak çevrilir.API Sorunları (Yavaşlık, 500 hataları, JSON olmaması): Standart try/except bloklarıyla yakalanır. Zaman aşımları ve geçersiz yanıtlar müşteriye 502 Bad Gateway olarak dönülür.Geçersiz Miktar (Amount): Eksik, sıfır, negatif veya 9 haneden fazla ondalığa sahip miktarlar 400 hatası ile reddedilir.
