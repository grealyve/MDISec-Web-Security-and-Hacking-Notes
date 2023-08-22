<h1 align="center">XSS Güvenlik Zafiyeti Part - 1</h1>

## Tespit
#### XSS HTTP response’un Body kısmında gerçekleşir. Buradaki dataya odaklan. Hedef, arkadaki sistem değil onu kullanan kişiler aslında.
- Bir HTTP response’u 3’e ayrılır:
    - CSS : Kaşının, gözünün, teninin rengini verir.
    - HTML : Senin iskeletindir.
    - JavaScript : Vücudunu hareket ettirmeni sağlar. Biz de hareket kısmıyla ilgileniyoruz.
- *Response’un içerisinde kontrol edebildiğin bir değişkeni(değerini kendin belirleyebildiğin), browser’a geri gönderilen HTML içeriğin bir kısmında kullanılıyor.*
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/3159ec38-3ae3-4ff6-9df9-b32f81f84359)
- Search kısmı olur, URL’in keywork kısımları olur, input edebildiğin her yer olabilir.

- Browser, gelen HTML’i parse ediyor ve bir tane structure oluşturuyor. JS ile ilgili kısımları JS interpreterine veriyor, CSS kodlarını çalıştırıyor ve ortaya bir tane DOM oluşturmuş oluyor.
    - Inpect yaptığında ve sayfanın kaynak kodlarına baktığında orada bulunana kodlar biribirinden farklıdır. Çünkü browser o kaynak kodundaki JS kodlarını falan işliyor ve ortaya site çıkmış oluyor. Biz inspect kısmı ile ilgilenicez.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/9352f9a4-105d-43f4-9191-16bf4450c9a9)
- Response’un içerisindeki HTML içerik browser, browser bir yazılımdır,için bir input kaynağıdır. Browser’a giden HTML içeriğindeki taglerin manipülasyonuyla uğraşmaktayız.
- XSS payloadımızı input olarak verdiğimizde(search kısmı bile olabilir bu), backend bunu input olarak alacak ve response içerisine data olarak bu payloadı eklemiş olacak. Request gittiği zaman web application bir view oluşturacak.
    - Browser bu payloadı çalıştırıp çalıştırmaması gerektiği hakkında bilgi sahibi değildir.

- XSS saldırısına input olarak şöyle başlayacaksın: `mdi'"><`
- yazıp dönen cevabın içinde mdi geçtiği taglere bak.
- Browser tarafına bunun data olarak kullanılması gerekirken, tag olarak kullanabilme imkanı ortaya çıkıyor.
```html
<script>alert(1)</script>
```
- data olarak gönderdiğimiz verinin, browsera aynı şekilde geri dönmesi halinde burada anlamalısın ki < tagi browser için başka bir anlam ifade ediyor.

### Hikaye şu şekilde ortaya çıkıyor:

- Bu gönderilen yazı backend tarafında data olarak kalıyor ama tekrar browser’a response olarak geri döndüğünde browser bunu artık tag olarak yorumluyor ve XSS doğuyor.
### XSS Neden bu kadar tehlikeli?
- Xss ile pencere içerisinde yapamayacağın şey neredeyse yoktur. Tüm fonksiyonları çağırabilirsin. Resmen adamın mouse, klavyesini kontrol edebilir hale geliyorsun.
- Beef adında opensource XSS exploitation frameworkü var. Burada kurbanın kamerasını kullanarak fotoğraf çekmeni sağlayan fonksiyonlar bile var, chrome extensionu kurdurtabiliyorsun, screenshot alabiliyorsun...
    - Bu aracı kullanarak cookie’leri her zaman çekemeyebilirsin. HttpOnly diye bir mevzu var; JS kodlarının Cookie’yi manipüle etmesini engelliyor.
    - Bunlardan korktuğumuz için XSS çıkmasını istemiyoruz.
- MDI bir e-ticaret sitesinde XSS bulduğunda eğer URL checkout sayfası ise, kredi kartı ödeme formunun formidsini alıp, oraya girilen tüm formid’lerini çeker.
- Mesela facebookta bir XSS çıktığında, adam gelip zafiyeti tetikletip bütün chat historyini çekebilir.
- XSS denildiği zaman aklına Client side gelmesi gerekiyor. Hikaye hedef sistem değil, onu kullanan insanları hedeflemekten geçiyor.
- XSS payloadın her zaman response’da yansımaz. Yansıyorsa —> *Reflected XSS*
- Başka kullanıcıları da etkilemek istiyorsan URL shortener sitelerini kullanarak tetikletebilirsin. Yani olay şu: bir kullanıcı bu linke tıklıyor ve hedef websitesine(bir e-ticaret sitesi olabilir) gidiyor. Browserın başı kel mi giderken de bütün cookieleri ekliyor. Tüm cookieleri eklediği için o web sitesi için sen Mehmet D. İnce kullanıcısısın. Sonrasında bu adam bir script çalıştırmak istiyor, requestini atıyor, response Body’si içerisine de bu XSS payloadı yerleşmiş oluyor. Bu XSS payloadı ile istediğin şeyi yapabilir hale geliyorsun.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/5159870d-1588-48d9-bfb9-1d5b14188797)

- XSS payloadın her zaman response içerisinde yansımaz, DB’e kaydedilebilir.  Diğer kullanıcılar da bu veriyi DB’den çekip browser da bunu yorumlayınca sömürülebilir. Böylece Stored XSS ortaya çıkar.(*Stored XSS*)
    - Başka bir web servisi tarafından alınıp başka bir web application kullanıyor da olabilir. Yani payloadın gezmeye başlıyor. Bu sefer bu siteyi kullanan herkes etkileniyor, çok daha tehlikeli.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/0e672841-d238-429b-9e06-90a450a0988e)
- Browserlar “textarea” içerisindeki scriptleri çalıştırmaz. Senin de bu scripleri nereye yazacağın çok önemli.
    - Saldırı kodunun textarea’yı kapatma ile başlaması gerekiyor.
- Kendinden başkasına etki ettiremiyorsan *Self XSS* olmuş oluyor
- Web sitesinin başka bir web servisinden gelen veriyi encoding yapmadan getiriyorsa orada Stored XSS çıkabilir. Facebook örneği (1:32:00)
#### Sadece form alanına girdiğin bilgilerle XSS yapmayı düşünüyorsan XSS konusunda başarısız olursun. Facebook örneğinde dropbox ile dosya yüklediğinde adam dosya ismini encoding yapmıyor ve XSS çıkıyor orda.

## HTML Context
- <> tagların olabilmesi için bu işaretlerin olması gereklidir. Bunlar olmazsa asla XSS yapamazsın.
    - Encode edebilirler &lt; ve &gt; olarak.
## Attribute Context
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/a861aa0d-e76c-405a-a5b3-f178ea0e41f3)
- Bu tagin value attribute’unda olduğun için XSS oluşmaz(Mehmet kısmı.)
- Öncelikle “ koyarak bu attributeun tanımını bitirmelisin. Sonrasında kendi tagini açıp scriptini yazabilirsin.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/15ceac42-7800-4363-8cac-3b35bede2f2b)
- Encode edilse bile kendi attribute unu oluşturup script çalıştırabilirsin :
- <> encode edilmeli VE “ ‘ ` encode edilmeli!!
- Encode ediliyorsa zaten input tagi olduğu için aşağıdaki payloadı kullan:
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/5cbe533f-a7ea-4f31-9e7b-29fb357580f8)
#### Not: Internet Exlporer version 8-9’da HTML Parser Engine ` ` bu işareti görünce tag’i sonlandırıyor. 
## JSINLINE Context
- id kısmına alert(1) göndermen yeterli olur XSS tetiklemek için.
    - Her şeyi encode etsen bile bu XSS çalışır. Burada parantezleri encode etmen gerekiyor
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/6aba78d3-fa64-42d9-84a0-82ccfd88f1d7)
## HREF Context
- a href içine USER_DATA durum ile karşılaşırsan gidip JS’in browserlarda protocol handler olduğu durumu tetikleyebiliyorsun.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/478fa3bb-a215-4cf5-a196-c102bbcc6648)
## Mitigation
Nasıl çözülür? : Input Validation VE Output Encoding
Input’u encode etmek yanlış bir yaklaşım olur. DB’e kaydederken datayı bozmuş olursun veya başka bir ekip bu datayı kullanmak için decode etmesi gerekir.

#### Context Based Encoding Owasp
- Normalde sen böyle encode etmezsen sadece Browser’ın temelde yaptığı encodingleri sağlamış olursun. Hangi contextler için hangi karakterler encode edilmesi gerekiyor ise bunuları encode eden yardımcı methodları var bunlar kullanılmalı.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/38ae5316-e42b-4841-bf66-6deed96c189e)
---
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/ab4612ea-0467-477d-b264-8043bc66b74a)
- Burada Asla XSS oluşmaz ama gidip yazılımcı Command line yaparsa işi biter.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/dad18be3-14c4-448c-b2de-ee07ac4134c5)
- %0a new line demektir ve hiçbir Template Engine whitespace karakterleri default olarak encode etmez ve XSS payloadımızı istediğimiz gibi yazabiliriz.
- Bu yüzden context base encoderları anlamak ve öğrenmek önemli.
## Google XSS
### 2.Challenge
- Blockquote içinde script çalıştıramayacağın için svg tagini dene. Olmazsa img denersin.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/8213b73a-76aa-480b-8995-0a8cac772012)

### 3.Challenge
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/2145b7e1-7065-4a45-8fcd-11a98bddf9d7)

- Bu kodun arkada yaptığı şey; location hash üzerinden gelen veri hiçbir zaman backend’e gitmez. Yani backend XSS’in yaşandığını. Ama JS buradaki location hashi input olarak kullanmakta.
- Burada bir XSS ortaya çıkıyor:
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/0ab8c886-e663-479d-8d95-1d5e96572721)
Payload kodu:
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/e3f3d582-4faf-428a-b93c-9a427101ae62)

### 4.Challenge
- Kullanıcının inputunu şu şekilde tutuyor —> inputa  = ’-alert(1)-’ yazdığında tetiklenecektir.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/af2e354d-8283-4c78-89a3-69e5fc10e9cc)
- startTimer içerisinde DOM’u manipüle edebilen bir fonksiyon kodu görmedik. Şu yapı çalışacaktır:
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/5b77d479-1ffa-489d-b033-edbbecf7a195)
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/af7bd7e8-a79d-47f3-a75f-8196cf090f9c)
#### NOT: HTTP içerisindeki + ifadeleri özellikle Query Stringde HTTP’nin ilk satırını GET’in bozduğu için boşluğu encode etmek gerek, + kullanmıyoruz da - kullanıyoruz.

### 6.Challenge
- Location hashden bir path alıyor sonra “includeGadget” ile bu pathi yüklüyor. Eğer verilen input http ile başlarsa hata verir. http isteklerini göndermeni engelliyor kod. 
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/f21e2e33-6e06-4423-a0b7-e00b66c17086)
```javascript
data:text/javascript,alert(1)
```
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/dce770eb-3013-4592-b56d-4096d8441a7f)
- URL’deki # burada js’in kodu anlamlandırmasını sağlıyor. 

### Ekstra Notlar:
- DOM based xsslerde hangi fonksiyonların engellendiğini filtrelendiğini kolayca anlayabilmek için url kısmında kendi fonksiyonunu yazıp bakabilirsin
