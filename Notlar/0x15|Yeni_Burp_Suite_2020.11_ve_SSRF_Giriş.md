<h1 align="center">SSRF Giriş</h1>

#### İşinize yarayabilecek bazı eklentiler: hackvertor, hackbar
#### Okuyabileceğiniz makaleler:
1) https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery#bypassing-filters
2) https://www.blackhat.com/docs/us-17/thursday/us-17-Tsai-A-New-Era-Of-SSRF-Exploiting-URL-Parser-In-Trending-Programming-Languages.pdf
3) https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery#bypassing-filters

## Lab: SSRF with blacklist-based input filter
- Bir ürüne gidip “Stock Check” yap. Parametre olarak bir URL gönderiyor bunu da decode et.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/42a7c449-28cd-4b22-82b6-f9bca3aab2ff)
- Öncelikle götürmek istediğimiz domaini çözmesi gerekiyor vatandaşın.
### Deneyebileceğin Yaklaşımlar:
1) Blacklisting yaklaşımlı SSRF çözmek için ilk olarak DNS çözdürmeyi deneyebilirsin(localhostpentest.pentes.blog/admin).
2) İp adresini HEX karşılığında verebilirsin ya da domaini HEX karşılığında verebilirsin.
3) Portu yazarsan arka tarafta backending URL parser’ı ile ilgeliniyorsun demek.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/12ef7c40-2e28-4545-8bcd-cc8dbc923342)
- [localtest.me](http://localtest.me) yapmasının sebebi arka tarafta 127.0.0.1’e çözümlemesini sağlamak. Bunu başka bir sürü kodla da yapabilirsin(nslookup ile de sağlamasını yaparsın.) github linkinde var.
- Burada anlamaya çalıştığı şey, string.contains() metodu gibi bir şey mi var arkada ona göre güvenlik önlemi almış yoksa URL’i parse edip mi bakıyor?
  - bunu anlamak için “admin”i alıp http://localtestadmin.me/ yapıp öyle deneyince security reasons hatası geldi yine demek ki burda string kontrolü var.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/7098af0b-cea5-41a4-ac42-ecce0f3a84df)
- ADMIN’i büyük yazıp denedi, “CouLd not connect to external check service” hatası geldi. Case sensitive mi diye kontrol etti yani
- “admin” i 2 kere URL encode edip yolladı. İlkinde öndeki engeli atlayacak admini göremediği için, bu URL’i ikinci encode halinde de bir backend’e(Java) tekrardan HTTP requesti göndertecek. Ama şuanki backend Java galiba ve bu isteği atarken otomatik decoding yapmıyor bu yüzden hata alındı.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/baded3ac-deed-46eb-a2f8-60ce283b8668)
- İlk decodingi Nginx, apache gibi önde duran uygulamalar yapar. Sonra 2. web uygulamasının gördüğü değer de encoded bir şey, burada bulunan nginx, apache, tomcat gibi uygulamalar tekrar bunu decode ediyor.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/28e062dd-19ff-4150-898f-2e098355abef)
- 127.1 yazınca oldu. gerisi /admin encoded şeklinde.
- Sonra carlosu tekrar istek atarak sildi.
## Lab: SSRF with whitelist-based input filter
- Whitelisting yöntemleri : domain şu olması gerekli, port olarak bu olmalı ve path olarak da bu olmalı diyerek sıkılaştırılabilir.
- Bunu atlatmanın da tek yöntemi backend’in URL parser function’ını buglaman gerekli.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/63e2a996-23b8-4af7-89a2-f8e0b49b375b)
- Normal şartlarda x:y# kullanıcı adı parola demek URL şemasında. Ama # den sonrasını location hash olarak görüp bundan sonrasını kale alma diyebilir. Burdaki logic bugı kullanman lazım. URL parser stock.weliketoshop . net i domain sanacak ama full URL’i HTTP kütüphanesine verince ben z.com’a gidicem benim domain bu diyecek, oraya da localhost’u yerleştireceksin.
- Hangi parametreleri whiteliste almış onu dene.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/b9ff47aa-95ec-4145-b292-424de3507715)
- Parametresini (storeId) silince kızıyor. & URL encoding yaptı.
- productId yi de görmek istiyor
- pathi değiştirince kızmıyor, admin yazınca kızmaz.
- Port’u değiştirince “CouLd not connect to external check service” hatası verdi yani buna da kızmıyormuş.
- domain’i değiştirince kızıyor. [stock.weliketoshop.net](http://stock.weliketoshop.net) olmalı diyor. sonuna .asd falan yazınca anlaşılıyor bu. URL’i parse ettikten sonra comparasion yapıyor
- Parse olayını daha detaylı incelemeye al.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/9b85c23e-58d6-4a69-ae91-fd2a75b88dcb)
- username:paswd kısmına burp collabrator koydu, pek mantıklı bir yaklaşım değil ama kızmadı yine.
- boş @ işareti yolladı. invalid URL döndü
- @ işaretinden önce # koydu, kafası karışmaya başladı.
    - URL parse edip denedi.
- 2 tane @ koydu.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/29c37c89-e50d-41dd-ac61-2355168f09fb)
- Protokol ile de ilgilenmiyor arkadaş
- Diyez # koyunca olmuş dendi, ama kafası karışıyor.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/f999b8de-32bc-4808-8b5f-a18156b70ca4)
- @ den sonrasını parse etse, whitelist hatası verecek zaten.
- ![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/867d2742-10a7-4b4d-828e-2d4974bb293d)
- Buna fuzzing yapıyormuş reis. Ama Parser kızdı zaten. Sitede, altındaki tabloda fuzzingi göstermiş zaten
- ![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/3e8d2bf2-6881-4911-86fe-4e3a3e7df803)
- Double encoding yaptı / işaretine ve oldu.
  - Payload, 127.0.0.1:80/admin@stock…..
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/1a371af7-920f-4db3-94a8-fcf92a98eef6)
- Parser @ den öncesini kul adı ve parola olarak görüyor yani çalışıyor o kısımda sıkıntı yok. Ama / işaretini koyunca burdan sonrasi itibariyle benim için path’dir diyor URL parser. 127.0.0.1 i algılamış oluyor.
- Öndeki URL parser diyor ki @ den öncesi kullanıcı adı, parola ben @ den sonrasına giderim aga.
- Sonra tam URL HTTP req. yapan fonksiyona gidince, URL decoding yapıyor ki parametreleri görebilsin. HTTP librarylerinin GET metodları default olarak URL decoding yapar. Sonra decode olmuş tam URL arka tarafa gidiyor o da bir kere daha decoding yapınca **/admin** ile karşılaşıyor.
  - Normade bunun böyle çalışmaması lazım çünkü @ işaretinden sonra orayı path algılayıp kafası karışmalı. Bunu çözümü de /admin?@ şeklinde yazmaktır ki ? den sonra query string olsun ve geri kalan her şey GET parametresine dönüşsün. ? ini de encode etmen lazım.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/ca7e59c1-9710-4b3e-9983-b2a41094550e)
- Özetle, HTTP web server bunu decode etti ve şuna dönüştü:
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/5d5d0fec-082a-47f4-97b0-7b12ac235354)
- backend(/product/stock endpointi)
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/7cd1e92f-d622-4682-a8df-be90da00f114)
- URL’i parse ettiğinde, /admin? encoded olduğu için 127.0.0.1:80 kullanıcı adı, paroladır. stock.weliketoshop.net benim domainimdir. Bu URL’i HTTP Client Library’sine verdi ve destination URL olarak bunu SET etti. Bu kütüphaneler de URL’den hangi host’a gitmeleri gerektiğini tespit etmek için bir daha URL parsing yaparlar ama yapmadan önce decode ederler ki “query stringleri “bulabilsin. Decode edince de parser, /admin ile karşılaşıyor ve destination 127.0.0.1 e dönüyor. ? sonrasını query stringe çevirmek için önemlidir.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/5a19a934-8cf9-4ae4-b08a-41fa8cad99fa)
- :80/admin/delete falan o kısımları sildik çünkü diğer /admin parametresi admin sayfasını getiriyor. Ona da /delete parametresini ekleyince ve username’i de en sona eklediğinde çözülmüş oluyor.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/1bf20385-ec3d-4d06-9884-ffdfc2581290)
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/58370d88-789e-41bb-807f-7e306e957ec2)
- Öndeki uygulamaya giden payloadı şu şekilde görüyor:
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/50cf111e-21ba-4eb3-a129-2584745b9cea)
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/ffefce34-8ddb-4662-9a77-e69f4f6bdc9e)
- 127.0.0.1 den sonra / koymayınca işler değişiyor:
- ![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/b1ac0c62-11ca-48c6-8e2f-140e41d68965)
