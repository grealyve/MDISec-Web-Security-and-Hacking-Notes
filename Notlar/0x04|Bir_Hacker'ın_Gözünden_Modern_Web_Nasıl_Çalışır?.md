<h1 align="center">Bir Hacker'ın Gözünden Modern Web Nasıl Çalışır?</h1>

#### Not: İlk kısımları bildiğim için not almayıp atlamıştım. İlerleyen zamanlarda ekleyeceğim.

- Bilgisayar ilk açıldığında 192.168.1.0 broadcast yapıyor ve ip adresi istiyor. DHCP Protokolü ile alıyor ip adresini.
- Bilgisayarlar birbiriyle konuşmak için MAC adresini öğrenecek bunun için Layer 2’de ARP protokolünü kullanıyorlar ve OS ARP Table oluşturuyor.
- Bilgisayarı açtığında DNS’i ve Gateway’i manuel olarak girmiyorsun. Bunları DHCP sağlıyor.
- Bir ip adresinin LAN içinde olduğunu anlaması Subnet Mask sayesinde oluyor. 255.255.255.0. Bilgisayar konuşmak istediği ip ile subnet mask ip’nin bit karşılıklarını yazar 1-1 işleme sokar ve çıkan sonuca göre karar verir.

- [www.google.com](http://www.google.com) ‘a gitmek istediğinde
- DNS’e koşmadan önce bilgisayar “*Host*” dosyasına bakmalıdır. /etc/hosts burada bulamayınca DNS’e soracak:
    - DNS’e(8.8.8.8) UDP protokolüyle 53 numaralı porta x.com’un ipsi ne? - Resolver DNS
- Resolver DNS bilmiyor ise, Root DNS’e gider sorar. O da bu ip adresinin kim olduğunu bilmediğini ama nereden öğrenebileceğini söyler yani Top Level Domain’e yönlendirir(TLD *.com/.net/.org).
- TLD de x.com’un kayıtlarını tutan adama yönlendirir. Authoritative DNS (Yetkili DNS) e yönlendirir. Yani kurumun DNS’i sanırsam.
    - dig NS google.com

##  Ne Gibi Riskler Var?
- Başka biri 8.8.8.8’e gittiğinde bütün süreç tekrar yaşanmayacak çünkü TTL’lere göre Cache’de tutma mekanizması var.
    1) Bu Cache’i poison edebilirsen x.com’a gidecek HTTP paketlerini istediğin gibi yönlendirirsin.
    2) Authoritative DNS sunucusunu ele geçirirsen, MX kayıtlarını değiştirirsin bütün e-postaları üstüne alırsın, Cname ağ recorlarını değiştirirsin tüm kayıtları üstüne alırsın, .txt kayıtlarını değiştirirsin sertifika issue ortaya çıkar.
    3) TLD saldırıları sıkıntı, tüm *.com cevaplarını yanlış döndürebilir sana.

## Bir IP ile Konuşmak
- Ip adresinin 80 portuna TCP SYN paketi yolluyor üçlü el sıkışma tamamlanıyor...
SYN —>
    <— SYN + ACK
ACK —>

![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/3e39b107-2cd6-4e9c-92d2-fae4b61b8f0f)
- Şimdi artık HTTP yollayabilirsin. 1 HTTP GET ve response aldığında aşşağıdaki katmanlarda 150 tane istek gidip geldiğini görebilirsin ve bu da HTTP’nin boktanlığından kaynaklanır.

## Virtual Hosting (VHOST)
- Domain sayısı ip sayısını geçtiği için böyle bir şey ortaya çıkmış.
- Host firmasının konfigürasyonuna göre hangi siteye gidiyorsan sana /var/www/… sitesine yönlendirir.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/41ed88da-da31-470e-b527-ddf077bf9eeb)
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/3e675dff-f89b-44c6-bfb5-df8df554ac7e)
- Web app. sayısı artarsa ve gelen requestleri dağıtmak için ön tarafa reverse proxy(LOAD BALANCER) konursa içeride sessionu sunucu ile senkronize etmen gerekecek. Yani diskte tutulmaması gerekiyor DB’de tutulabilir.
    - DB içinde tutulmasının da dezavantajları var. Yük ve performans konusu devreye giriyor.
    - Birden fazla DB varsa SQL Proxy gerektiriyor. Bunun da yerini Mikroservisler alıyor(?)
    - Session oluştuğunda sunucu tarafında diskte tutabilirler ama bu diskte IO işlemine sebep olur. Sistemi yavaşlatır yani.
        - Kullanıcı her istek attığında sen bu cookie’yi alacaksın
        - diskteki hangi session hangi session folder’ı bununla alakalı bulacaksın dosyayı okuyacaksın
        - Session’a bir şey yazdığında dosyayı update edeceksin
- Bütün web app. lerin statik dosyalarda aynı içerik olması için CDN(Content Delivery Network) de gerekiyor. Cloudflare gibi programlar ile bir dosyanın ya da HTTP response’unun kolaylıkla yapılabilmesi sağlanır. Mesela bir resimin yüklenmesi, Farklı ülkelerdeki kullanıcılara teker teker sunucu isteği atılıp gönderilmez de Cloudflare üzerinden(burada Firewall ve Reverse Proxy ikilisi oluyor) otomatik gönderilir. Tüm trafik bu CloudFlare üzerinden akıyor, Çin’deki bir adama da resimin aynı cache değeri dönüyor yani sadece güvenlik değil mesele. Senin evinde yaşanan olayların aynısı içerideki sunucuda da yaşanıyor.
- Full text searchleri veri tabanı üzerinde yapılması sonucu ortaya çıkan maaliyetleri engellemek için : Elastic Search
- Sunucuya gelen bütün HTTP isteklerinin “Loging” ini tutan sunucular bulunur.
- Bu sunucunun da aynısının yedeğinin bulunduğu bir sunucu daha olur : DRC
    - Olur da sistemin çalıştığı data center patlarsa tüm trafik hiçbir kesinti yaşanmadan DRC hizmet verir.
- SSL Reverse Proxy kısmında sonuçlanıyor. HTTP requestini işleyip inceleyen adam reverse-proxy. HTTP Desync açığı çıkabiliyor sunucu başka görüyor, RP başka görüyor.
