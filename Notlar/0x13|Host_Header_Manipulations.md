<h1 align="center">Host Header Manipulations</h1>

- Domain sayısı > IP adresi, bu yüzden shared hosting diye bir mevzu var. Apache, tomcat, nginx gibi web servislerinin önünde bulunan reverse proxy servislerinin tüm ilgi alanı HOST ile ilgilidir.
- Atak yüzeyleri de arkadaki frameworkler olmaya başlıyor.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/1945dc61-200d-4aec-9951-24c0e86f45bc)
- Arkadaki frameworkler URI veya URL yapısını istemektedir.
  - Request.Url.ToString() metodu gibi fonksiyonlar framework tarafından çokça kullanılır. Özellikle statik dosyalarının full URL’lerinin template engine lerde generate edilmesi ile alakalı mevzu dönüyor.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/0d75e742-2c22-4814-a094-60e41e56753e)
  - Bu yardımcı metodlar HTTP requestinin içindeki host headerının içindeki bilgiyi alırlar.
- Ön tarafta reverse proxy i aşacak, arkada tomcati aşacak, bu iki noktadan sorunsuz geçtikten sonra burdaki requesting Map edilmesini sağlarsan ve bu uygulamada Host alanını kullanarak bir şeyler print ediyorsa XSS ortaya çıkar.
- Hatta uygulamadan kullanıcıya dönen HTML içerik, Cache mekanizmalar tarafından cache leniyor ise böylece aynı içerik birden fazla kullanıcıya sunuluyor ise sen Cache’i poison ederek Stored XSS çıkartabilirsin
- Nginx, Host’a göre uygulamaları map ettiği için sen gidip localdeki uygulamaları yazarsan, yanlış konfigüre edilmiş bir reverse proxy i sömürürsün dışarıdan erişimi olmayan bir uygulamaya erişebilme imkanı sunulmuş olur.
## Lab: Basic password reset poisoning
- Uygulama bir e-posta göndereceği zaman bir HTML’i render eder. HTML’in içerisinde full URL oluşturması gerekmektedir.
- Parola sıfırlama alanının host kısmı buradan alınır.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/5a4b7832-7044-4fb0-9fd8-4cf4c3e1239e)
- Buradaki domain alanını manipüle edip, kendi domainimiz ile değiştirmek istiyoruz. Böylece bizim web siteye gelicek ve tokenı da elde etmiş olacağız.
- Önemli: Bir web uygulamasının testing, paging süreçleri falan farklı domainlerde gerçekleşebileceği için hard coded yazamaz bu host alanını, o yüzden dinamik bir şekilde oluşturulmalı. Framework üstünde de ona göre oluşturulur.
- Biz de bu şifre sıfırlama isteğini proxy ile kesip, Host alanına kendi domainimizi yazıcaz.
- Normalde carlos dış dünyadaki domainimize gelebilir, ama bu labda öyle tasarlamamışlar kendi exploit server oluşturmuşlar onun üstünden yürünecek.
- Adam linke tıkladığında Access logdan toplayacaksın veriyi.
- Adamların oluşturduğu exploit serverının sonuna ? koyup Host’a yazdığında loglara düştüğünü göreceksin. Normalde Host kısmında / falan geçmemesi lazım.
  - ? işaretini ise, geri kalan kısmı query string’e dönsün, adamların sunucuda kafalar karışmasın diye. Normalde bir doman yazarsın yürür gidersin.
  - Router ile farklı controllera gitmesin diye.
## Lab: Password reset poisoning via dangling markup (kapanmamış tag, havada kalmış)
- Bu labda adam token falan vermiyor, gidiyor yeni password veriyor.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/a655e745-3a2e-464d-b417-c34a5a31a828)
- Domain kısmına öyle bir şey yazmamız gerekiyor ki “href”, parolayı taşıyan bir URL’e dönüşsün.
  - Click here’a tıkladığında otomatik gidecek oraya zaten, linki görmüyor.
- önce
  - ‘></a> tagını kapatıyor
  - img tagi açıyor ve “ koyarak başlatıyor ama kapatmıyor, oradan sonra gelen her şeyi e-posta istemcisinin HTML Parser’ına oynuyoruz ve o gösterecek bize. (e-postayı alan şey, outlook)
  - tek tırnak işaretini beğenmiyor bu iş iptal, URL parserlara gidecez
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/add34ec6-9156-4c90-abf7-28ebd93d37c8)
- Ön taraftaki reverse proxy(gateway) host alanını parse ediyor, ondan sonra applicationda bu host alanını kullanıyor.
  - URL parse olayını kral bildiği için :80’ koyup  deniyor ve sonuç 200 dönüyor. Heralde :80 den sonrasına bakmıyor. : olan kısma kadar domain olarak kabul edip ona göre MAP ediyor.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/d08381e3-629a-4533-bdeb-18aff11af4cd)
  - Arka taraftaki Web app bu Host taginin tamamını kullanacağı(FULL URL IDENTIFIER) için sonrasında kendi payloadımızı çakabiliriz.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/332f3889-83f4-45eb-a3c8-c0e9a726bf0f)
  - Burada context based encoding yapması lazım.
- Sonrasında loglara parola düşüyor.
