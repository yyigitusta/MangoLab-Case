# Notes

One page is plenty. Four short sections:

## Decisions

Aldığım en kritik mimari karar, eksik tarihlerin (özellikle hafta sonları veya bugünün kurunun henüz yayınlanmadığı saatler) yönetimiydi. Kullanıcı bugünü sorduğunda eğer API veri yok deyip 404 dönerse, müşteriye hata fırlatmak yerine şeffaf bir şekilde `/latest` (en son yayınlanan kur) verisine dönmeyi tercih ettim. Müşteriyi yanıltmamak için API'den gelen gerçek yayınlanma tarihini `rate_date`, müşterinin asıl sorduğu tarihi ise `asked_date` olarak JSON yanıtında kesin bir şekilde ayırdım.Bu kararı almamda B sorusunda yazılan kod etkili oldu.

## With another day

Daha fazla geliştirme yapmam gereken validasyonlar olabilir. Testler sonrası bu eksiklere ulaşılabilir.
## AI tools

Hızlı prototipleme yapmak, `pytest` test ortamını kurmak ve `respx` kullanarak internetsiz (mock) test senaryolarını hızlıca inşa etmek için bir yapay zeka asistanı kullandım.Ayrıca syntaxına alışkın olmadığım bir dili yazmamda yardımcı oldu
## One thing the AI got wrong

API'ye istek atacağımız kodu yazarken, yapay zeka parametreleri ` params={"from": from_, "to": to} ` şeklinde isimlendirdi. Bu, dış API'nin 404 (Not Found) fırlatmasına sebep oldu çünkü Frankfurter API'si parametre isimlerini ` base ` ve ` symbol s` olarak bekliyordu. Swagger UI üzerinden manuel test yaparken bu sorunu fark ettim ve parametre anahtarlarını düzelterek yönlendirme sorununu (routing issue) çözdüm.
