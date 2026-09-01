# Review of tool.py

One page. Findings **ranked** — most harmful to a customer first.

For each finding: what is wrong, what it does to a customer (not to a linter),
and how you would verify it.

## 1.
Ne yanlış: Bellekteki önbellek sözlüğü (_cache) anahtar olarak yalnızca f"{base}-{target}" değerini kullanıyor ve istenen tarihi (on) önemsenmiyor.

Müşteriye etkisi: Bir müşteri geçmiş bir tarihe ait kur istediğinde (örneğin 2020 yılı), bu kur önbelleğe alınır. Hemen ardından başka bir müşteri güncel kuru istediğinde, sistem güncel kur yerine önbellekteki eski tarihli kuru döner. Bu durum, ödeme yapan müşterilerin tamamen yanlış finansal hesaplamalar yapmasına ve doğrudan maddi zarara uğramasına yol açar.

Nasıl doğrulanır: Eski bir tarih için kur isteği at (örn. on=2015-01-01), ardından tarih belirtmeden hemen güncel kur isteği at. İkinci isteğin de eski 2015 kurunu döndüğünü görebilirsin.

## 2.
Ne yanlış: except Exception as exc: bloğu tüm hataları yakalayıp konsola yazdırıyor ve HTTP 200 (Başarılı) statü koduyla birlikte rate: 0.0 ve result: 0.0 dönüyor.

Müşteriye etkisi: Geçersiz bir para birimi girildiğinde veya dış servis (upstream) çöktüğünde, yapay zeka modeli müşteriye hata bildirmek yerine "250 EUR, 0.0 TRY eder" der. Model ve müşteri bu tamamen sahte ve sıfırlanmış rakama güvenerek yanlış işlem yapar.

Nasıl doğrulanır: Olmayan bir para birimiyle istek at (örn. to=XYZ). Servisin hata kodu dönmek yerine HTTP 200 ve sıfır değerleri döndüğünü test et.

## 3.
Ne yanlış: rate_date alanı, dış API'nin döndüğü gerçek tarihi (payload["date"]) okumak yerine, kullanıcının sorduğu tarihi veya bugünün tarihini (str(on or date.today())) baz alıyor.

Müşteriye etkisi: Dış API bir yedek kur döndüğünde (örneğin Pazar günü için Cuma gününün kuru), araç bunu sanki Pazar gününe aitmiş gibi etiketler. Kullanıcı/model verinin geçerlilik tarihi konusunda yanıltılır.

Nasıl doğrulanır: Geçmiş bir Pazar günü için istek at. JSON yanıtındaki rate_date alanının, dış API'nin Cuma gününe ait olan gerçek tarihini yansıtmadığını, yanlışlıkla Pazar gününü gösterdiğini kontrol et.

## The one I would fix before shipping tonight
Önbellek anahtarına tarihi de dahil etmek (f"{base}-{target}-{on}"). Geçmiş kurları güncel kurmuş gibi ödeme yapan müşterilere sunmak, güven sarsıcı ve doğrudan maddi kayba yol açacak kadar büyük bir veri bütünlüğü hatasıdır.
Ayrıca validasyon işlemlerinde müşteriyi bilgilendirecek dönüşler sağlanmalı.

## Things that look suspicious but are fine
Ben herhangi bir sorun bulamadım (Kod çalıştırmayan) ancak ai ile sohbetimde from anahtar kelimesinin rezerv olmasından dolayı from_ parametre adlandırmasının sorun olabileceğini söyledi.
