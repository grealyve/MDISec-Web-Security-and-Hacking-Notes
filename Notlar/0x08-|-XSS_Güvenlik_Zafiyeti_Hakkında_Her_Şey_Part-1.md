<h1 align="center">XSS Güvenlik Zafiyeti Part - 1</h1>

## Tespit
#### XSS HTTP response’un Body kısmında gerçekleşir. Buradaki dataya odaklan. Hedef, arkadaki sistem değil onu kullanan kişiler aslında.
- Bir HTTP response’u 3’e ayrılır:
    - CSS : Kaşının, gözünün, teninin rengini verir.
    - HTML : Senin iskeletindir.
    - JavaScript : Vücudunu hareket ettirmeni sağlar. Biz de hareket kısmıyla ilgileniyoruz.
- *Response’un içerisinde kontrol edebildiğin bir değişkeni(değerini kendin belirleyebildiğin), browser’a geri gönderilen HTML içeriğin bir kısmında kullanılıyor.*
![1](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/8713127e-3a56-4e9f-b61f-5e72998a8da1)
- Search kısmı olur, URL’in keywork kısımları olur, input edebildiğin her yer olabilir.

- Browser, gelen HTML’i parse ediyor ve bir tane structure oluşturuyor. JS ile ilgili kısımları JS interpreterine veriyor, CSS kodlarını çalıştırıyor ve ortaya bir tane DOM oluşturmuş oluyor.
    - Inpect yaptığında ve sayfanın kaynak kodlarına baktığında orada bulunana kodlar biribirinden farklıdır. Çünkü browser o kaynak kodundaki JS kodlarını falan işliyor ve ortaya site çıkmış oluyor. Biz inspect kısmı ile ilgilenicez.

- XSS saldırısına input olarak şöyle başlayacaksın:
```html
mdi'">< 
  yazıp dönen cevabın içinde mdi geçtiği yerlere bak.
Browser tarafına bunun data olarak kullanılması gerekirken, tag olarak kullanabilme imkanı ortaya çıkıyor.
<script>alert(1)</script>
```
- data olarak gönderdiğimiz verinin, browsera aynı şekilde geri dönmesi halinde burada anlamalısın ki < tagi browser için başka bir anlam ifade ediyor.

- Bu gönderilen yazı backend tarafında data olarak kalıyor ama tekrar browser’a response olarak geri döndüğünde browser bunu artık tag olarak yorumluyor ve XSS doğuyor.
- XSS payloadın her zaman response’da yansımaz. Yansıyorsa —> *Reflected XSS*
    - DB’e kaydedilebilir.  Diğer kullanıcılar da bu veriyi DB’den çekip browser da bunu yorumlayınca sömürülebilir.(*Stored XSS*)
    - Başka bir web servisi tarafından alınıp başka bir web application kullanıyor da olabilir.
![2](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/888e5082-851c-4625-ae89-900af82fe2fd)
- Browserlar “textarea” içerisindeki scriptleri çalıştırmaz. Senin de bu scripleri nereye yazacağın çok önemli.
    - Saldırı kodunun textarea’yı kapatma ile başlaması gerekiyor.
- Kendinden başkasına etki ettiremiyorsan *Self XSS* olmuş oluyor
- Web sitesinin başka bir web servisinden gelen veriyi encoding yapmadan getiriyorsa orada Stored XSS çıkabilir. Facebook örneği (1:32:00)

## HTML Context
- <> tagların olabilmesi için bu işaretlerin olması gereklidir. Bunlar olmazsa asla XSS yapamazsın.
    - Encode edebilirler &lt; ve &gt; olarak.
## Attribute Context
![3](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/70bde275-8a01-4c28-ac51-b98f6869c7bc)
- Bu tagin value attribute’unda olduğun için XSS oluşmaz(Mehmet kısmı.)
- Öncelikle “ koyarak bu attributeun tanımını bitirmelisin. Sonrasında kendi tagini açıp scriptini yazabilirsin.
![4](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/3a176077-2b3a-4bd3-b9f7-7a540719ee38)
- Encode edilse bile kendi attribute unu oluşturup script çalıştırabilirsin :
    - <> encode edilmeli VE “ ‘ ` encode edilmeli!!
![5](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/b8757f62-e8cf-4267-921a-30f617276ea3)
#### Not: Internet Exlporer version 8-9’da HTML Parser Engine ` ` bu işareti görünce tag’i sonlandırıyor. 
## HREF Context
- a href içine USER_DATA durum ile karşılaşırsan gidip JS’in browserlarda protocol handler olduğu durumu tetikleyebiliyorsun.
![6](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/f4a2f59d-0764-4413-a6b0-af917af8b163)
## JSINLINE Context
- id kısmına alert(1) göndermen yeterli olur XSS tetiklemek için.
    - Her şeyi encode etsen bile bu XSS çalışır. Burada parantezleri encode etmen gerekiyor
![7](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/046cd8ca-aa2c-460e-9ab8-520fa934cdb0)
---
![8](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/3edf18d0-7d8a-4afd-a1a9-a7020feb38d9)
- Burada Asla XSS oluşmaz ama gidip yazılımcı Command line yaparsa işi biter.
![9](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/51d9ac18-43c5-4765-9985-6fe8eab3f496)
- %0a new line demek
## Google XSS
- Blockquote içinde script çalıştıramayacağın için svg tagini dene. Olmazsa img denersin.
![10](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/94cc6588-cdbd-47c7-9a74-be3ae5745b18)
- Kullanıcının inputunu şu şekilde tutuyor —> inputa  = ’-alert(1)-’ yazdığında tetiklenecektir.
- HTTP içerisindeki + ifadeleri özellikle Query Stringde HTTP’nin ilk satırını GET’in bozduğu için boşluğu encode etmek gerek, + kullanmıyoruz da - kullanıyoruz.
![11](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/30e9027d-7b13-43fb-bced-8c1a2c5447b4)
-----
- Location hashden bir path alıyor sonra “includeGadget” ile bu pathi yüklüyor. Eğer verilen input http ile başlarsa hata verir. http isteklerini göndermeni engelliyor kod. 
![12](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/72148484-b37e-4839-8860-ca12b0392960)
    - data:text/javascript,alert(1)
![13](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/a2ac1aa8-ed6e-4b90-a8d3-be2ec89d5ca6)
- URL’deki # burada js’in kodu anlamlandırmasını sağlıyor. 

### Ekstra Notlar:
- Nasıl çözülür? : Input Validation VE Output Encoding
Input’u encode etmek yanlış bir yaklaşım olur. DB’e kaydederken datayı bozmuş olursun veya başka bir ekip bu datayı kullanmak için decode etmesi gerekir.
Context Based Encoding Owasp
- DOM based xsslerde hangi fonksiyonların engellendiğini filtrelendiğini kolayca anlayabilmek için url kısmında kendi fonksiyonunu yazıp bakabilirsin
