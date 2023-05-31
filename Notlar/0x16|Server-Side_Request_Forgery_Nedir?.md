<h1 align="center">Server-Side Request Forgery (SSRF)</h1>

### Araştırabileceğin konular: 
1) Path normalization vuln 
2) Response header injection
### Okuyabileceğin bir makale: 
- https://medium.com/techfenix/ssrf-server-side-request-forgery-worth-4913-my-highest-bounty-ever-7d733bb368cb
## SSRF Nedir?
- Sunucu’nun bir isteği olması lazım, sunucu’nun bir kaynağa ürettiği istekten bahsediyor. **Server’ın client gibi davrandığı işlerin tamamı.**
  - Third party bir API’a ulaşıyor olabilir, senden alınan bir URL’den erişim sağlıyor olabilir, birtakım farklı servislere erişim sağlıyor olabilir.
- SSRF konusunda, sunucu bir adrese HTTP talebi göndermekte sen de bu HTTP talebini uygulamanın kendi ağında bulunan sistemlere yönlendirtmesini yapmaya çalışıyoruz.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/3ff9ddb5-d922-4424-9bd7-ccfd28e3a92c)
- Eğer bu uygulama bir Cloud servisi ise, AWS, Azure, Google cloud gibi; ***AWS metadata curl*** gibi bir olay karşına çıkıyor.
  - Cloud servislerinin içerisindeki her instance’ın erişebildiği sabit bir endpoint bulunmaktadır.
  - Bu adrese request oluşturduğun zaman AWS içeriden sana cevap veriyor. İçerideki bilgileri dızlayabiliyorsun yani.
  - Service account tokenlarını elde edebilirsen, diğer instance’ları queryleyebiliyorsun.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/84e1c279-66b6-4995-8875-c88a645499bd)
- Sunucu, kullanıcıdan aldığı inputu valide eder ki onun içerideki servislerine erişemesin diye. Burada da 2 yaklaşım söz konusu:
  - Blacklisting
  - Whitelisting
- Uygulamanın external bir resource’a erişimini abuse ediyoruz.
- Blind SSRF: External resource dan gelen cevabı uygulama gitti DB’e ya da diske kaydetti. Sana sorgunun cevabını göstermiyor. Alıyor doğrulama yapıyor bir şey yapıyordur ya da web-hook gibi triggerdır. Alıp isteği atıyor o kadar işi bu.
- Ya da belli condition altında tetiklenen kurallar belirlenmiştir.
- Millet bunu RCE’ye çevirmek için uğraşıyor ama RCE ye çevirmek için başka bir zaafiyete de ihtiyaç duyuluyor.
- XXE gibi external bir soruce’a isteği attırabiliyorsun yani kafana otursun. XXE zaafiyetinde de SSRF ve genellikle Blind SSRF’e dönen bir hikaye oluşuyor.
- Adam koyar bir dockera(HARDENING YAPMIŞTIR) senden aldığı URL’i ve docker da internete istediği gibi çıkıyordur ama iç ağıyla iletişim kuramıyordur öyle kalırsın. Ama burda da farklı atak vektörleri de ortaya çıkabiliyor. HTTP Curl gibi uygulamanın kendisi içerisinde blduğun memory corruption zaafiyetlerini de sömürebilmektesin.
- Mesela başka bir zafiyeti kullanarak gittin bu uygulamanın localdeki IP adresini buldun, 10.0.10.2:x/ bu IP adresi üstünde hangi portlar var diye kontrol et, intruder ile daya bunu. Networkün içerisinde de yapabilirsin bunu.
- XXE zafiyeti ile networkte bulunan bir ElasticSearch tespit etmiş, direkt protundan cevabı almış.
## Lab: Basic SSRF against the local server
- Bir üründe Check Stock yapınca, uygulama bir URL adresine HTTP isteği gönderecek. Arkadaki backend bir proxy gibi davranıyor, senden bir istek alıyor ve onu başka bir yere yönlendirmesini yapıyor.
- Uygulamanın üstündeki localhost’a yönlendirtme yapmaya çalışmaktayız.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/8cfc7b4c-eb99-477c-bcf8-845e0907169c)
- [localhost](http://localhost) yerine istediğimiz herhangi bir şeyi yazabiliriz.
- Bu uygulamanın üstünde çalışan bir ElasticSearch olsaydı tüm endpointlerine gidebilirdin. Ama POST metodu istiyor olabilir bu endpointler. Sunucu da senin verdiğin inputu GET methodu olarak yolluyor olabilir. Bu tür limitasyonlar ortaya çıkıyor.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/b0894206-f3a9-4834-a0dd-f8ea645ba444)
## Lab: Basic SSRF against another back-end system
- 192.168.0.x:8080/admin ip ve portunda bir açık varmış. X ne ise onu tespit edip ona göre çözeceksin.
## Lab: Blind SSRF with out-of-band detection
- Referrer headerı’ı üstünde bir zafiyet varmış. Gidip oraya [google.com](http://google.com) yazdığında istek gidiyor 200 OK dönüyor. Sonra bunu collaborator ile deneyince çıkan DNS talebini tespit edebiliyoruz. DNS’i çözmeye çalışıyor yani.
- HTTP istediğimiz adrese çıkıyor ama cevap sana dönmüyor.
## Lab: SSRF with filter bypass via open redirection vulnerability
- Bu uygulama sunucusu 302 Redirection gördüğünde bunu takip ediyor mu etmiyor mu böyle bir hikaye var.
- Application aldığı URL üstünde validation yapıyor, ama validation’ı hangi katmanda yapmakta ne şekilde yapmakta? Localhost’a erişim sağlayamazsın diyor mesela sen de gidip burada Open Redirect zafiyeti tespit ediyoruz.
- Uygulamanın kendi URL’inde bir adrese gittiğinde senin kontrol edebildiğin bir yere Redirection yapıyor. Bu durumda redirectionlarda validation sağlıyor mu???

- Adam artık akıllanmış URL almıyor, sadece endpoint alıyor ve ben oraya giderim diyor.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/fa8f96e7-cfa7-4a59-9d1f-a489445c442a)
- Kendi ana sayfasına git diyorsun yani / koy.
- /afasfashf Not found
- Bu URL’lerin birinde zafiyet  bulsak, bu zafiyet de Open Redirection zafiyeti sağlasa.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/e00f3eed-82e2-44de-bd18-53b37291ed17)
### Bu aslında path normalization ile de çıkabilirmiş??  /product/stock/check…;
- Login sayfasını denedi, giriş yaparak redirection bakıyor sanırım
- Ürünlerde “next product” diye bir buton varmış. ProductId’sine redirection sağlıyor. Bu bilgiyi de path’den alıyor, bu path üstünde HTTP isteği sallayabiliriz.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/1aa9b03c-b21a-445c-885d-c3f75ff228be)
- Gereksiz parametreleri silmek gerekiyor. “?currentProductId” gibi
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/1a0d21df-b2a4-45f3-88a6-40cf6489e31f)
- Sonra stockApi parametresi üstüne yazıcaz bunu. Bu path, redirection yapıyor ama adam da bunu kontrol etmiyor(validation sağlamıyor).
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/c85aa403-3a27-4fdf-9f9d-9f4784dd2a0e)
- stockApi bunu blacklisting yapıyor olsaydı “nexProduct”ı validation yapıyor olsaydı, giderdi şu istek üstünde uğraşırmış. Admin kısmında new line edip header injection yapabilirsin.
- Bu olay validation’ın yapısına bağlı.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/08869694-4e33-41e3-b302-3f0fc1cd142f)
- Response header injection
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/432d311c-4c7e-49bf-a5b9-5acc40ed90b2)
## Lab: Blind SSRF with Shellshock exploitation
- Burp’ün “Collaborator Everywhere” diye bir extensionu varmış. Her giden requesti manipüle ediyor, tüm olası headerın içine yazıyor. “Guess Header”ı vuruyorsun aga.
- Elle buldu, Referer alanına yazdı bunu. Sonra Collaborator’den baktı gelen HTTP isteğe User-agent Mozilla olduğunu gördü. İçeride Command line’dan chrome çalıştırıyor olabilir. Host alanına çünkü collaborator adresi gelmiş.
  - asd:afas@collaborator yazdı gelen isteğe baktı sonra, iplemediğini gördü.
  - Shellshocku görünce ayıktı, User-agent’ı bizden alıyor. eben yazınca collab. gelende de eben yazıyordu.
- Böyle bir şey yaptı, id yerine whoami.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/8c05f942-1f86-4241-a4fa-e03690109b6e)
- Port da belli değil ama hayırlısı. Multi-step exploitation yapacaaaaaaaaz.
- Uygulama, Referer’dan aldığı URL’e HTTP talebi gönderirken User-agent kısmına da senden aldığını yazıyor. Tüm iç networkü scan edecek. İç networkteki bir web server cgi, shellschock zafiyeti var.
- Shellshock zafiyeti, User-agent üzerinde cgi larda tetiklenebilen bir zafiyet.
- **whoami komutunu bir subdomain olarak dışarı taşıyor.**
  - curl, wget komutları ile de yapılabilir
- Önce zafiyet hangi makinede onu tespit edicez. curl,wget gibi komutların full pathini yazmak gerekiyor olabilir. O pathi tespit etmeye çalış collab. ile.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/b7eebe63-2024-40a7-b79d-a1e56bb87129)
- Portu :8080 yaptı oldu 🙂
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/af37bf6d-0afe-41c5-a893-12d82844f9a2)
