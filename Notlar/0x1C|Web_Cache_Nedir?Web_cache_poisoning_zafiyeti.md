<h1 align="center">Web Cache Poisoning Zafiyeti</h1>

## Web Cache Nedir ?
- Cache dediğinde birçok kişinin aklına “Browser Cache” ve “DNS cache” gelir.
  - DNS cache: işletim sisteminizin cache yapması ve DNS serverının da cache yapmasıdır TTL değerleri falan vardır.
  - Browser cache: ctrl+f5 :D
- Server-side caching, modern frameworklerin template enginelerinin sağladığı caching yöntemleri.
  - Laravel, related template engine var. Her seferinde template sıfırdan üretmektense bunu bir ara cache formatında tutar ve diske kaydeder. Lazım olduğunda ise cache den alır ki aynı işlemleri ve maaliyetleri tekrarlamamak için.
- Bizim konuşacağımız template engineler ile alakalı değil. Web cache bunlarla alakalı değil spesifik olarak.

- İki kişi aynı sayfaya geldiğinde sürekli aynı şeyleri tekrar tekrar üretmektense bunu direct cache de tutarlar ve burdan serve edilir. Bunların hepsi HTTP katmanında oluyor.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/9cb4031d-50ca-4f38-8e61-bb839d1ec16b)
1) Bu sistemde hangi adreslerin cache leneceğine kim karar veriyor?
2) Bir requesting cache den mi döneceği yoksa sunucuya mı iletileceği nasıl karar veriyoruz?
- Her cache edilen şeylerin bir key-value değerleri olması gerekiyor. Value zaten sunucudan dönen content.
  - Key ise, web servisi kendisi üretiyor. Protokol, domain, path ve query stringi baz alarak üretiyor.
    - in memory çalışır bu DB falan kullanmaz.

- Elimizde böyle bir key yoksa gelen HTTP requestlerini sunucuya göndermek zorunda. Gider sunucu bir response üretir ve sonrasında sunucu cache ile ilgili esktra bir bilgiyi ortadaki Web-cache servisine söylemesi lazım. Bunu da header valuelar ile söyleyebiliyor. Cache=True
  - Bu content cashlenebilir mi cashlenemez mi onu söylemesi lazım /private-key ya da /settings gibi her usera özgü farklılık gösterecek endpointlerdir bunlar.
  - Query stringler çok kritiktir bu cache mevzusunda.

- Bir istek gönderiyoruz, Serverdan Web-cache mekanizmasına dönen cevapta bizim isteğimiz cache lenebilir ise buraya(response un body kısmına) bir şey injekte etmeyi başarabilirsek ve benden sonraki gelen kişinin contentin içeriğine istediğimizi yazabiliyoruz.
  - Peki ya request gönderdiğinde sana gelen cevabın Web-cache mekanizmasından dönüp dönmediğini nasıl anlayacaksın?
### “              Göremiyorsan delay koy.          “
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/df48e41e-96c4-4a5f-80be-837984d9566e)
### İLK TENKİK : query stringe rastgele bir değer yaz. İlk gönderdiğinde ve ikinci gönderdiğinde zaman farkı varsa anlayabilirsin.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/4722fc3a-8c6b-437d-8c73-2fbc6d0ef231)
### İKİNCİ TEKNİK : Cache sunucular sana söyler.  OK ya da MISS yazar. Bazıları cache keyini yazar(kısa bir key). 
- Responsedaki headera bakarsın 200 OK 404 fark etmez.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/de0aef41-db7f-48d5-93eb-448cbd694a27)
- Bir endpoint parametresi almıyordur yani ?selam=kelam yaz yolla
  - Caching yaparken spesifik bir query stringe bakmaz sistemler.

## Response Contentine nasıl bir şeyler injecte edeceğiz? (Lab: Web cache poisoning with an unkeyed header)
- Query stringler key’in belirlenmesi için anahtarlanıyor burada.
  - Key value - Unkeyed value  diye 2 tane hikaye var. Key’de kullanılan valuelar var, keyde kullanılmayan valuelar var. Query stringin neredeyse tamamı keyde kullanılır. Burda caching yapmak çok zordur. Çünkü onu değiştirdiğin zaman key değişecektir, başka bir kullanıcının contentine injection yapamazsın.
- Modern frameworkler işin içerisine giriyor burada. Ana sayfaya gelen herkes aynı cachei görmektedir. Yazılımcı açısından debugging için bunu görmesi gerekebilir falan ama bu zafiyet açığı değildir.
- Cache poisoning için arka tarafı da biraz bilmek gerekiyor.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/732669bb-d962-428e-bffc-673ecf2e892f)
- Template engine 2.satırı alıp aslında 5.satıra çeviriyor. Peki buradaki protokol, domain, port nereden geliyor?
  - Modern frameworkler bunları hesaplarken Request Headerından yararlanırlar.
    - Mesela http isteği geliyorsa protokol olarak onu yazar, X yerine gider Hostu yazar, port tanımı olmadığı için otomatik 80 alır. Link üretirken bunları kullanırlar işte.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/6c3908c5-ce07-42dc-82b3-b1fd41569258)
- Mesela gidip host alanını ordan direkt çekiyormuş. (Değiştirmeyi denedik başaramadık abi)
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/2822e6fe-cd89-48b0-b27e-264cade67c4c)
- Host alanına dokunamadık, ön taraftaki Nginx Host tanımına göre çalışır ama arka taraftaki web application framework X-Forwarded-Host headerı yazarsan buna göre çalışır. Link üretirken helper functionları bunu baz alabilir mesela.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/37d205e9-767b-4fb2-a6fd-18ff68e617f7)
- X-Cache : miss yazıyor ama tekrar isteği yollayınca Hit oldu. Caching key üretirken  X-Forwarded-Host headerına  bakıyor demekki.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/db98c163-5035-46a6-8855-27c64bba50f2)
- Cache counterı belli bir sınırdan sonra keyi silebiliyormuş silmesi için sürekli denedik ama olmadı gibi.
- Sonra şunu repeaterda üstüne oynamak için aldık. **“Unkeyed bir value bulduk ve bu valuenun responsta görev aldığını gördük.”**
  - Temiz bir ortamda çalışmak için ?a=a yazıp o ortamda test yaptı:
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/e0138fc5-f6e9-4b08-b46e-2a34b4deeb6d)
- Böyle bir XSS payloadı injecte edebiliyoruz. Aynı cache keyini başkalarında hit ettirebildiğimizde Stored XSS olacak. Yani ana sayfanınkini de böyle yapmamız lazım. Yukarda da yazdığım gibi sürekli istek giderse o cache invalide olabiliyor. Burada query string içerisinde bunu yaptık ama zor gibi. 
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/4fc37c0b-0ab9-4380-b6e3-52c71207f52b)
- böyle yolladı sadece query stringi sildi. Ama arada bu injection kodu kayboluyor, sürekli bu paketi yolluyor ta ki Miss gelene kadar:
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/4c1e697f-c22d-4174-8c94-2b3f23b1db3f)
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/1ea2aca5-b07e-490a-926b-3ff9ee301c83)
## Gerçek hayattan örnekler
- Bazen Web-appler asenkron şekilde JS contenti generate etmesi gerekir dinamik şekilde.
  - User’ın sessionına göre alıp buradaki birtakım HTML içeriklerini değiştirmeniz gereken bir endpoint. Backendde bir kodu var. Dinamik bir single page sayfası olduğunu düşün.
      - Herkes home page e geliyor ve dönen content de JS. Sen Cache server tarafından keylenmeyen bir değer bulursan patlatırsın. SANA ÖZGÜN BİR CONTENT ÜRETMESİ LAZIM. Bu case olma olasılığı çok düşüktür.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/a21aee4b-c378-4652-b92b-3bd4f1b16d8a)
- tracking.js full path üstünden çağırılmış
  - // tricki ise senin isteğin http iste http, https ise https üzerinden çağırması için kullanılır.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/b0b5191d-2e92-4634-b93c-42801bd69306)
- Javascript contentini cache hitliyor. Buradan da o case çıkabilir.
  - Header kısmına X-Forwarded-Path : /selam yazdı bir cacık olmadı. X-Forwarded-Port : 1 yazdı olmaıd.
  - Extender “Param Miner” kuruyosun. Sağ tıkla guess header diyosun.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/05c74e23-a02d-4958-a13e-84af2c1ed135)
- real lifeta görsem geçerim diyor :)
