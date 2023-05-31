<h1 align="center">Directory Traversal</h1>

#### Bu iş RCE’ye gider
#### Bu zafiyetin çözüm noktası, okuyacağın dosyanın nihai path’inin istediğin destination ve aynı noktada olup olmadığına bakman.
#### Gerçek hayattaki uygulamalar filename inputunu alıp decode etmek gibi şeyler yapmazlar, URL decoding gibi. Senden alınan filename bir URL’in içerisine konup, backend bu URL’e HTTP req gönderip resim alan bir mekanizma varsa işte o zaman double encoding işe yarıyor.

# Bu zafiyetin varlığı nasıl tespit edilir?
- Bu adam bir path’de bulunuyor ama pathin ne olduğunu bilmiyoruz.
- Bir dosyada ../ yapıp sonra tekrar aynı dizine geldiğimizde sıkıntı olmadan dosya geliyorsa directory traversal zafiyeti var demektir.
  - **../image/24.jpg**
---
- Dizin gezinmece.
- Günümüzdeki web applicationları resource erişimleri sağlıyor. Diskte bulunan bir veriyi okuma gerçekleştiriyor.
- Zafiyet, web app. kullanıcıdan aldığı bir input ile yereldeki  bir dosyaya erişim sağladığı yerde sıkıntı çıkıyor.
    - Sen gidip o dizini değiştirip, daha sonra okuma, silme, değiştirme (I/O) işlemi gerçekleştirebilme.
    - file=x
- Elinde id=5 olan bir post var ve bunun içinde de bir resim var. Bu resmin full path’ini expose edersen, full path’e erişimi olan biri bu resmi görebilir. Resmin full path’ini de DB’de tutuyorsun onu kullanıcıya dönüyorsun, kullanıcı ona geldiği zaman gene Django’nun execution içerisine giren bir yapı yok yani.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/9e40249e-0716-491f-ac25-7cebcfc81bda)
- IMG resource’una HTTP talebi geldiğinde önde çalışan Nginx(reverse proxy) kuralı bunu resim olarak gördüğü için localden dosyayı okuyup sana geri sunar. Yani app. içindeki hiçbir kural permission mimarisi içerisine girmez. Doğrudan erişebilirsin.
- Bu yüzden  uygulamalar conten erişimi sağlarken DB’deki her şeye erişim kontrolü yaparken statik resource’lara bunu pek yapmıyorlar. O yüzden bir resim göstermek istediğin zaman reverse proxy kuralları ile proxy service olarak sunman gerekiyor ya da farklı modeller kuruyorsun.
  - S3 bucket içerisinde miniofile yapıları?!?
- Böyle bir fatura görme yapısı vardır 15’i 16 yapınca basic IDOR denersin ama yapamazsın. Gidip pdf’i export et dediğinde backend pdf üretip, web sunucusu içerisindeki diske koyar ve bunun linkini sana verir. Bu link de app. permission şeması içerisine girmez.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/d01119a1-981e-4e6a-8af2-ace03b9fda00)
- Arkadaki yapı şu şekilde
  - /var/www/hackerconf.stream/static/ahmet
  - ahmet’i okumaya çalışacak gidip.
  - ../../../../../../../../../../../../../../../etc/passwd Linuxta üst dizine çıkmanın sonu yoktur garanti olsun diye 500 tane koy.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/abe889ce-647b-4d5e-ac2e-c7c2683ec673)
- ~ vs. shell environment’da geçerlidir. Bir programın okuduğu path erişiminde her program bunu destekliyor olamaz. O yüzden home dizinine gitmek yanlış olur, home dizini belki yoktur vs  vs.

- Özellikle Load Balancer’ın yaptığı URL normalization ile arkadaki application serverın yaptığı URL normalizationlar farklı ise, ön taraftaki app. serverdan izniniz olmayan noktalara erişim sağlayabiliyorsunuz.
  - Uber’in SingleSignOn mekanizmasını bypasslayıp internal sistemlere erişim sağlamada. URL normalization ve parsing.
---
## Lab: File path traversal, simple case
- Gerçek hayatta böyle sorgular yapılmaz aslında.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/fa530990-0e16-4442-9e97-27f62ccfaf1a)
- Benden aldığı bir inputu bir file reading işlemi yapacak.
## Lab: File path traversal, traversal sequences blocked with absolute path bypass
- Arkadaki kod resorce contcatenation yapmıyor olabilir, direkt resource okuyor olabilir. Yani bu çalışan uygulama ile aynı dizinde bulunuyorsa gitmek istenen nokta, bu birleştirmeyi yapmaz. Üst dizine çıkmak bir şey ifade etmeyecektir bu yüzden full path vermek zorundasın.
  - /etc/passwd
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/cbb4827a-9c45-42f9-a1d0-9fdab5a9ab91)
## Lab: File path traversal, traversal sequences stripped non-recursively
- Genelde yazılımlarda ../ yapmana engel koyarlar.
  - ../24.jpg yaptığında hala aynı cevap geliyorsa demek ki ../ kaldırıyordur adam.
  - Bunun da çözümü ….// yapmandır
## Lab: File path traversal, traversal sequences stripped with superfluous URL-decode
- Input validation’dan mı hata alıyoruz yoksa dosyayı gerçekten bulamadığı için mi hata alıyoruz diye kontrol sağlandı.
- Gerçek hayattaki uygulamalar filename inputunu alıp decode etmek gibi şeyler yapmazlar, URL decoding gibi. Senden alınan filename bir URL’in içerisine konup, backend bu URL’e HTTP req gönderip resim alan bir mekanizma varsa işte o zaman double encoding işe yarıyor.
- Biz de URL encoding yaparak payload deniyoruz ve dizini de bulmaya çalışıyoruz.
    - GET / image?filename=..%25fimages%25f41.jpg
        - 2 kere encoding yaptı /
    - noktalara kızsa .jpg okumaz zati
- ./ taktiğini denedik, ../ konduğunda gene okuyor, ….// gene okuyor
- / görünce delleniyor oğlan
    - % nin encoded hali %25, üst dizine çık tekrar gir yaptı
    - %2f in decoded hali / (slash) işareti.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/ed164602-c2f1-4a5e-9703-1a28d738b468)
## Lab: File path traversal, validation of start of path
- Böyle bir şeylerle karşılaşıyorsun:
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/ec4497cd-da88-4d59-baf3-1657e5f41a53)
- Bazı durumlarda backend, başlangıçtaki dosya yolu ile validation sağlar.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/eb495769-b035-4a07-80db-36effc4a43c8)
## Lab: File path traversal, validation of file extension with null byte bypass
- Senden folder alır.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/120552e9-c258-49d3-b64b-d776bba7e9b8)
  - /var/www/hackerconf. stream/.. /.. /.. /.. /.. /.. /.. /. ./selam/liste. txt
    - SQLi yaparken # ile sağ tarafı kapatabiliyordun ama bunda yapamıyorsun.
  - PHP’de eskiden en sona %00 yazdığında string bitti sanıyordu sonradan düzeltmişler.
- Burada null byte çalışır, git en sona %00 koy.
- Ve lab sayfasında da .jpg uzantısını istediğini söylemiş
- Bunu yapmasının sebebi;
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/f966acfb-4aad-40c1-9c29-68eab6a3096e)
- String birleştirme yapacağı zaman adam, NULL Byte’dan sonrasını kopyalamıyr adam, zafiyetin oluştuğu yer burası.
## Directory Traversal ile ilgili zaafiyetlerden nasıl yararlandım ?
- Apache solr adlı yazılımı file adlı bir dosya alıyormuş. Logları okumak için bu kod parçacığını yazmış reis. Sonra cookie çalmış RCE’ye kadar yolu var.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/fc8dbaf9-affe-4c7e-9dca-ac5ec2c46b0e)
