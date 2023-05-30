<h1 align="center">IDOR (Insecure Direct Object Reference)</h1>

- Application securityde en önemli konu kullanıcıdan alınan inputlar, direktiflerdir.

# IDOR Nedir?
- Bir web sitesikullanıcıdan aldığı bilgiler ile veritabanına erişim sağlayıp bu veriyi okuyup kullanıcıya gösterme, veriyi değiştirme, silme gibi işlemler yapar. IDOR'un gerçekleşmesi için bazı kurallar gerekiyor örneğin: 
  - Ben başka bir kullanıcının adresini görememeliyim

## Uygulamalı Örnek
- Aşağıdaki burp ss i için: 
- Veritabanındaki kullanıcının ID’si 15 imiş. Bunu anlamış olduk. 15i 14 yapınca işler değişiyor. Forward etmeden önce Action kısmından “Do intercept response this request” yap ki sonucu görebilesin.

![1](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/1b091bd7-69ae-4b88-a26e-f388e085e448)
  - Veri tabanında adress bilgisini ifade eden id değeri 15, bu bizim başlangıç noktamız olacak.
- Normalde bir kullanıcı, başka bir kullanıcının datasını değiştiremiyor olması gerekiyor. Bu veriyi silme yetkisi var mı? Bu veri senin mi kontrollerini yapması gerekiyor?
- Gelen cevap 302 FOUND olabilir ama buna aldanma, authorization failure hatası da alıyor olabilirsin. Adres silinmiş de olabilir silinmemiş de olabilir. Gidip diğer kullanıcıyı kontrol et. 
- Sonra veritabanında var olması güç bir ID üstünde paketi yolla, sitenin backend davranışını anlamaya çalış. 
  - 404 Not Found döndürdü. ID validation yapıp ondan sonra silme işlemini gerçekleştiriyor. Eğer bu id yok ise adresi silmeyip 404 hatası döndürüyor. Kullanıcının silme yetkisini de kontrol ediyor bu arada. 
![2](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/2b1991f9-f3b8-4559-847f-f8d365221791)
- Delete fonksiyonu yerine belki arkada update fonksiyonu vardır onun üzerinde deneme yapıyoruz. CRUD(Create-Read-Update-Delete).
- Referans noktan arkadaki fonksiyon olacak. Intruder’dan ekleyeceksin buraya popüler fonksiyon isimlerinden gireceksin. “From field names”
- Başka bir fonksiyona eriştiğinde : Missing Function Level Access Control zaafiyeti var demektir. Intruder’dan “edit” isimli bir fonksiyona eriştik. Eğer bu endpointi kullanarak başka idler ile farklı verilere erişebiliyorsan IDOR zafiyeti demektir.
------------
- Adres kısmını bu uygulamada başka nerelerde kullanılıyor onu düşünmen gerekiyor.
- Sipariş verirken adress ID sini değiştirdiğinde ve Forward yapınca sipariş tamamlanırsa IDOR var demektir. Bu fotoğraftaki address verisi bizim 17 değerini tutuyor bunu değiştirip başka adrese yollayabiliyor musun?
![3](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/e0520726-b364-4356-a5cb-0cb085e91e9a)
- Adress ID değerini 18 yapıp yollayınca sipariş tamamlandı, başka birisinin adresine ürün gitmiş oldu. History kısmından ise başkasının adres bilgisine ulaşmış oluyoruz.
![4](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/1dfbd479-b258-4d15-a201-579ac9314e2b)
- Bu adres bilgisi bu user’ın mı? Bunun kontrolü unutulmuş bu yüzden IDOR zafiyeti çıkıyor. Sadee bu adres var mı yok mu onun kontrolünü yapmış.
![5](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/53e4c8a8-e54a-44a9-9b41-08f079b958b5)
- Zafiyetler 2 türe ayrılıyor :
1) Teknik zafiyetler (payload var bunlarda örn. SQLi, XSS)
2) Business Logic zafiyetler(kaynak kodda bulması da zor, örn. IDOR)

- Günümüzde artık olay microservicelere kayıyor olaylar. Bir uygulamada aynı DB üstünde kendi sorgusunu çalıştıran onlarca servis var. Birisi sadece kullanıcı adını, biri sadece adresini, biri sadece kullanıcı bilgilerini gibi… Burada business logic kurallar nereye konulacak? Yani hangi user’ın adresini görebileceğin kuralını nereye implemente edeceksin? Servis mi kontrol edecek yoksa bu servisi çağıran uygulama mı?
  - Eğer servis, yetkiyi kontrol ediyorsa şöyle bir sıkıntı olacak: yarın bir gün bu servisi kullanmak isteyen başka bir web uygulaması başka bir yetki ve servis yapısı istediğinde sen microservice deploy edeceksin. 
  - Servisin görevi verilen işi yapmaktır, iş nedir = adres bilgisini getir. Adresi görebilirsin, göremezsin derse “annotation” devreye giriyor(can read gibi).
![6](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/732978d1-69bd-4973-a441-49bbb606cccb)
- Ama illaki bu servis içinde kontrol edicem diyorsan “user_id”sini de alıyorsun ve SQL sorgusu içine ekliyorsun ve bu userın bu adrese erişip erişemeyeceği gibi “JOIN, WHERE” komutlarında yazması lazım. Eğer bunu unutursa parametreleri pass ediyor ama alt katmandaki servis kontrolü yapmıyor, servisteki adam bu yetkileri kontrol etmiyorsa IDOR oluşmuş oluyor. Büyük firmalarda genelde bu servisler ayrı takımlar tarafından kodlanıyor o yüzden dikkat.
Aşağıdaki ss için: 
- API Gateway ile Servis yazılım dilleri farklı olduğu için IDOR ortaya çıkıyor.
- API Gateway 5 üzerinden işlem yapıyor fakat servis dili ben son gelen parametreyi id olarak aldığı için “6 id” li kullanıcının bilgilerini döndürüyor.
    - Kimi JSON parserlar ilk gördüğünü değer olarak kabul ederken kimisi ikinciyi değer olarak görüyor.
![7](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/ddb3ca42-a210-4eba-9e79-dffb0f0ffaca)
- Web uygulamasının arka tarafında bir mikroservis yapısı varsa sen kullanıcı için cookie kullanmaya devam et. Ama API DB’e gidip user ile ilgili tüm yetki ve verileri toplayıp bir tane JWT oluşturup servislere bu JWT’yi yolladığında yetki kontrollerini servis üstünde çok daha hızlı yapabilirsin. Yani JWT oluşturup içeride onu gezdirirsin. Bu yapının da götürüsü olarak mesela kullanıcı başka bir web servisine gittiğinde JWT ile gelmediği için(cookie ile geliyor) bu API’lar için merkezi bir Database oluşturmak zorunda kalıyorsun, içeride JTW kullanmaya devam edebilirsin.
![8](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/d6ca5fef-642d-420b-9824-f7c690a199e9)

## Burp Suite ile IDOR'u Pentester Nasıl Bulabilir ? 
- Burp içinde Autmatrix extensionı kullanabilirsin.
![9](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/9a50299f-635e-49c0-8f7f-6146d1059d26)
- Yatay ve dikey senaryolar oluşturmalısın yani kullanıcıların birbirinin verilerini görüp görememe mevzusu başka; bir kullanıcı, adminin işlerine erişebiliyor mu bu mevzu başka. Her bir senaryoyu değerlendirmen gerekiyor.
- En az 2 farklı kullanıcı ile tokenları al Autmatrix içine koy. Bunun üstünde profiller oluşturup oynamalar yaparak IDOR zafiyetini tespit edebilirsin.
- Anonim user, normal kullanıcı 1, normal kullanıcı 2, moderatör ve admin gibi. Her bir kullanıcı için ayrı cookie değerleri girebilirsin.
- Bu extensionı kullanarak kullanıcıya dönen responseları filtreleyebilir, hangi kullanıcı için hangi response döndüğünü detaylı inceleyebilirsiniz. Kimi yerde 500 gördüğün yerde IDOR vardır , her zamna 200 dönmesi lazım değil.
- Anonim bir Cookie oluşturma : XSRF-TOKEN=invalid;SESSION=INVALID

IDOR Hakkında Her Şey: https://medium.com/@aysebilgegunduz/everything-you-need-to-know-about-idor-insecure-direct-object-references-375f83e03a87
