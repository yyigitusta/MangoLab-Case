# FX Converter Tool

Python ve FastAPI kullanılarak geliştirilmiş basit bir HTTP servisidir. Para birimi dönüşümlerini, `frankfurter.dev` üzerinden sağlanan Avrupa Merkez Bankası (ECB) kurlarını kullanarak gerçekleştirir.

Servis, bir yapay zekâ ajanı çalışma ortamında güvenilir bir araç olarak kullanılmak üzere tasarlanmıştır. Hassas para birimi dönüşümleri yaparken tarih bilgisinin de şeffaf bir şekilde ele alınmasını sağlar.

## Çalıştırma

Servis varsayılan olarak `8080` portunda başlar.

```bash
./run.sh

Farklı bir port kullanmak için PORT ortam değişkenini ayarlayabilirsiniz:

PORT=9000 ./run.sh

Testleri Çalıştırma
Test paketi pytest ve respx kullanır ve tamamen çevrimdışı çalışacak şekilde tasarlanmıştır.

./test.sh

Testler, upstream API adresini kapalı bir porta yönlendirir ve harici API yanıtlarını mock'lar. Bu nedenle testleri çalıştırmak için internet bağlantısı gerekmez.

Hata Kodları
Bir hata oluştuğunda API, 2xx olmayan bir HTTP durum kodu ve aşağıdaki bilgileri içeren JSON yanıtı döndürür:

Makine tarafından okunabilir hata kodu
Kullanıcı tarafından okunabilir hata mesajı
HTTP Durumu	Hata Kodu	Açıklama
400	invalid_input	amount gibi zorunlu parametrelerin eksik olması.
400	invalid_amount	Tutarın sıfır veya negatif olması ya da 9'dan fazla ondalık basamağa sahip olması.
400	same_currency	Kaynak (from) ve hedef (to) para birimlerinin aynı olması.
400	invalid_date_format	Tarihin YYYY-MM-DD formatında olmaması.
400	future_date	İstenen tarihin gelecekte olması.
400	date_too_old	İstenen tarihin serinin başlangıcı olan 1999-01-04 tarihinden önce olması.
400	currency_not_found	Belirtilen para birimi kodunun mevcut olmaması.
502	upstream_timeout	Upstream API'nin 5 saniyelik süre içerisinde yanıt vermemesi.
502	upstream_error	Upstream API'nin 500 gibi beklenmeyen bir HTTP hatası döndürmesi.
502	upstream_unavailable	Upstream API'ye ulaşılamaması veya geçersiz JSON döndürmesi.

Kenar Durumların Yönetimi
Endpoint, hiçbir zaman kur uydurmamak ve kullanıcıya kullanılan veri hakkında şeffaf bilgi vermek üzere tasarlanmıştır.

İstenen Tarihte Kur Bulunmaması
ECB her takvim günü için kur yayınlamaz. Hafta sonları, resmi tatiller veya o günün kurunun henüz yayınlanmamış olması nedeniyle istenen tarih için kur bulunmayabilir.

İstenen tarih için kur bulunmadığında upstream API 404 döndürür. Servis bu durumu yakalar ve bunun yerine /latest endpoint'inden mevcut en güncel geçerli kuru alır.

Şeffaflığı korumak için yanıtta iki ayrı tarih bulunur:

asked_date — kullanıcının istediği tarih.
rate_date — kullanılan kurun ECB tarafından gerçekten yayınlandığı tarih.
Böylece kullanıcı, dönüşümün istediği tarihe ait kurla mı yoksa en güncel geçerli kurla mı yapıldığını anlayabilir.

Gelecek Tarihler ve Serinin Başlangıcından Önceki Tarihler
Aşağıdaki tarihler için istekler FastAPI doğrulama katmanında doğrudan reddedilir:

Gelecekteki tarihler
1999-01-04 tarihinden önceki tarihler
Bu durumlarda upstream API'ye hiçbir istek gönderilmez.

Kullanılan hata kodları:

future_date
date_too_old
Para Birimi Doğrulaması
Kaynak ve hedef para birimleri aynıysa istek doğrudan reddedilir:

same_currency

Bir para birimi kodu mevcut değilse upstream API 404 döndürür. Servis bu yanıtı yakalar ve kullanıcıya HTTP 400 ile birlikte şu hatayı döndürür:

currency_not_found

Upstream Hataları
Upstream API istekleri 5 saniyelik timeout ve exception handling mekanizması ile korunur.

Aşağıdaki durumlar kontrollü şekilde ele alınır:

Timeout: httpx.TimeoutException oluştuğunda upstream_timeout döndürülür.
Beklenmeyen HTTP hatası: raise_for_status() ile yakalanan 500 gibi hatalar upstream_error olarak döndürülür.
Geçersiz JSON: API geçersiz veya bozuk JSON döndürürse upstream_unavailable döndürülür.
Upstream'e ulaşılamaması: Bağlantı hataları upstream_unavailable olarak döndürülür.
Tutar Doğrulaması
amount parametresi istek sisteme girdiği anda doğrulanır.

Aşağıdaki değerler reddedilir:

Eksik amount → invalid_input
Sıfır amount → invalid_amount
Negatif amount → invalid_amount
9'dan fazla ondalık basamağa sahip amount → invalid_amount
Bu doğrulamalar, para birimi hesaplamalarında kontrollü ve öngörülebilir bir hassasiyet sağlar.

Tasarım İlkeleri
Servis aşağıdaki temel prensiplere göre çalışır:

Kur uydurmaz — tüm dönüşümler upstream ECB tabanlı veri kaynağından alınan kurlar kullanılarak gerçekleştirilir.
Tarih bilgisinde şeffaftır — kullanıcının istediği tarih ile gerçekte kullanılan kurun tarihi birbirinden ayrı tutulur.
Erken doğrulama yapar — geçersiz istekler gereksiz upstream çağrıları yapılmadan reddedilir.
Upstream hatalarını kontrollü yönetir — timeout, HTTP hataları, geçersiz JSON ve bağlantı problemleri tanımlı API hatalarına dönüştürülür.
Test desteği sağlar — test paketi mock'lanmış upstream yanıtları kullanarak tamamen çevrimdışı çalışabilir.
