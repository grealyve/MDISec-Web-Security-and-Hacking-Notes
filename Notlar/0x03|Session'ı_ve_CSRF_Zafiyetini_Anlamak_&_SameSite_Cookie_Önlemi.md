<h1 align="center">Session ve CSRF Zaafiyeti</h1>

# Protokollerin Durumu Nedir?
- TCP handshake karşıdaki kullanıcıyı doğrulamanı sağlıyor. Karşındaki insanı doğrulabiyor olması en büyük artısı.
- HTTP’nin en büyük eksikliklerinden biri karşındakini doğrulayamıyor olman.
- HTTP’nin diğer en büyük eksikliği State/Stateless durumları.
- HTTP’de önemli bilgiler “header” kısmında gider; host, cookie, bağlantının devamı gibi… Body kısmında data gider.
![1](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/4801502d-ca58-4a88-beda-c895913a6c81)
- Her paket gittiğinde kullanıcı adı ve parola istememesi için cookie yöntemi kullanılıyor.

## Cookie Mekanizması Nasıl Çalışıyor?
- **Bir Websitesi neye göre Cookie Set ediyor?  —→ http://www.mdisec.com:80/**
    - **Protokol**
    - **Domain(subdomain dahil değil yani değişse cookie düşer)**
    - **Port**
![2](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/4cf52833-df92-4661-a4bd-762e0acc7f3f)
- path : Bu cookie’nin hangi sınırlar içinde kullanalıabileceğini belirler.
- Cookie değeri senin “session”a ulaşman için gerekli anahtar. Cookie’yi alan Reverse Proxy senin hangi sunucuya(1,2,3)’e gitmen gerektiğini yönlendirmesini yapıyor.

- Cookie oluşturmak adına başlattığın sessionlar /tmp/disk’e yazılır.
    - Session : Cookie’nin bilgilerinin yazıldığı ve tutulduğu bir bilgi topluluğu. %90 sunucu tarafında gerçekleşir yani clientin kendi tarafında değiştirebileceği bir şey değil.
1) Session oluştuğunda sunucu tarafında diskte tutabilirler ama bu diskte IO işlemine sebep olur. Sistemi yavaşlatır yani.
    - Kullanıcı her istek attığında sen bu cookie’yi alacaksın
    - diskteki hangi session hangi session folder’ı bununla alakalı bulacaksın dosyayı okuyacaksın
    - Session’a bir şey yazdığında dosyayı update edeceksin
2) Web app. sayısı artarsa ve gelen requestleri dağıtmak için ön tarafa reverse-proxy(LOAD BALANCER) konursa içeride sessionu sunucu ile senkronize etmen gerekecek. Yani diskte tutulmaması gerekiyor DB’de tutulabilir.
    - DB içinde tutulmasının da dezavantajları var. Yük ve performans konusu devreye giriyor.
    - Birden fazla DB varsa SQL Proxy gerektiriyor. Bunun da yerini Mikroservisler alıyor(?)
    - Bütün web app. lerin statik dosyalarda aynı içerik olması için CDN de gerekiyor. Cloudflare gibi programlar ile bir dosyanın ya da HTTP response’unun kolaylıkla yapılabilmesi sağlanır.
3) Content Delivery Network: Hiçbir diske dokunmayan, işletim sistemi aracılığıyla memory’de key-value şeklinde veri tutan servis. CDN
    - Memory’de tuttuğu için çok hızlı oluyor.
    - Integrity’ye güvenemiyorsun. CDN patlarsa en kötü 302 Redirection alır.
    - Bunun da backupını alman lazım.
4) Cookie Based Session : Client tarafında tutmak. Cookie yi json olarak veriyor server.
    - **Yani adam sana kul. adı parola ile geldi sen bunu aldın doğruladın ve bu adamın bilgileri ve permissionlarıyla bir obje oluşturdun, bunu SYMMETRIC ENC ile şifreledin(şifrelerken application settings tarafında set ettiğin secret keyi kullandın), sonra bunun HMAC ile signiture’ını aldın, adamın eline bunu verdin. —> Single Sign On bu mantıkta çalışıyor.**
    - SUNUCU TARAFINDA CONFIG DOSYASINDA BULUNAN SYMMETRIC ENC SUPER SECRET KEY ..!
    - Set-Cookie : SESSION={’IV’:’RANDOM_DEGER’, ‘session_data’: [’email’, ’user_id’]} | CHECKSUM
        - [’email’, ’user_id’] = ENCRYPTED
        - [Hiçbir verinin değişmediğini garanti etmek için]CHECKSUM = HMAC( {’IV’:’RANDOM_DEGER’, ‘session_data’: ENC}) | + KEY
            - Signiture olmuş olur elinde bunu da BASE64 ile encode edersin.
    - Bu mevzunun geliştirilmiş hikayeleri var : Openid, OAuth aynı mantık üzerinde ama farklı şekilde çalışıyor.
## CSRF (Crosss Site Request Forgery) Zaafiyeti
1.Tab içinde 18.132.45.78 sitesinde login olmuş kullanıcı var | 2.Tab’da hacker.com açık.
![3](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/79e4b29a-8ad9-4655-ad2d-6ffd56ca9e78)
- hacker.com’a gittiğinde bu resmi yüklemek için o http isteğini göndermek zorunda. O webb sitesine GET talebi gönderiyor, browser da Cookie’yi otomatikman ekliyor.Domain, protokol ve port koşulları sağlandığı için Cookie buraya da set edilecek.
- Browser, kullanıcının bu talebi isteyerek mi oluşturduğunu anlamak zorunda.
    - CSRF token oluşturup sunucuya göndermesi gerekiyor.
![4](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/1d33b33b-5e33-4c6b-b237-e72c12206865)
- Bu token bu session’a özel üretir.
- Bu tokenı http://18.132.45.78/address/delete/17/?_token=alskhfasuklhfasjlfasjdhask 

- Bu CSRF token bilgisi nerede tutuluyor?
    - Session nerede tutuluyorsa orada tutulur. Fakat token’ı illaki sessionda tutmak zorunda değilsin. User’ın kendi Cookie’sinde de tutabilirsin.
- REST API kullanılırsa?
    - Cookie diye bir şey olmaz. Origin diye bir şey olur. Doğası gereği CSRF zaafiye olamaz burda.
![5](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/b7cd572c-2e99-4ae6-830a-84665a14c97c)
Autharization diye bir header’ın olacak, oturum anahtarını burada taşıyacaksın
