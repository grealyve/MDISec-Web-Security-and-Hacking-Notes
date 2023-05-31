<h1 align="center">Server-Side Template Injection</h1>

#### Template engine’ler sandbox yapısına sahiptirler genelde ve burda çalışırlar.
- Uygulamalarda MVC(Model View Controller) yaklaşımı işin içerisine girdiği zaman, elimizde bir controller() var ve bunun da return(view) edeceği bir view var.
  - View yeni bir atak vektörü oluşturmuş oluyor bize.
- HTML implementasyonu değil de, bir template implementasyonu yapmaya başlıyoruz. Özelleşmiş HTML gibi.
- Backend DB’den veya kullanıcıdan gelen bir inputu name olarak buraya yerleştiriyor. Burda bir ***dinamik kullanım*** oluşuyor.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/87e456b8-9e03-4343-a2ae-fad06147a5e9)
- Ama bu konu ise, dinamik olarak bir HTML oluşturmanı gerektiren bir konu. Mesela sahibinden’de ilanların açıklama kısımları gibi. Ya da “sendgrid” gibi.
  - User’dan template’in kendisini alıyorsun artık.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/200e3fde-3b3a-4136-b112-e284bcc47dab)
- Bu template’in içerisinde bir obje varsa ve fonksiyonları çağırmaya başlarsa neler olabilir? Sunucu içerisindeki fonksiyonlara erişirsin belki? 
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/1fb66c9d-13d1-4bfd-8505-2b861b3d206c)
- Kaynak koda erişimimiz olmadan nasıl tespit edebiliriz?
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/fb806f3a-e5de-48a4-a7f5-7932809607f6)
## Lab: Basic server-side template injection
- Zafiyetin nerde olduğunu tespit edebilmek için gezindi. Sonra bir ürün listeleme isteğine baktı, 1.satıdaki parametreyi kurcalamaya başladı.
  - Geçerli olmayan bir integer değer girdi “Not found” hatası geldi.
  - String ifade yazdı, bu sefer de invalid hatası geldi. Demek ki buna göre kontrol ediyor.
- Sonra gitti 1 yazdı productId olarak. 302 döndü ve follow deyince bir sayfa geldi, parametre olarak da bizden “Unfortunately… “ mesajı almış ve cevaptaki sitede de bu mesajı yerleştirmiş.
- İlk olarak aklına reflected XSS gelebilir.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/31eb873d-f40f-481d-a202-3b3b74e8e077)
- Template’e bizden gelen parametreleri koyuyor olabilir django, laravel gibi kütüphanelerde render methodu oluyormuş. Arkadaki Template Engine’i tespit etmek için ise, XSS taglarinin peşinden
  - { {xx} } % {yyy} %${zzz}
  - Ya da { {7*7} } deneyip sonuçta da 49 olarak görmeyi beklersin. Kesin olarak Template Injection var diyebilirsin.
  - Bunun aynısı geliyor ise birtakım problemler var demektir.
- Bilmediği bir template engine var. Lab sayfasında ERB template yazıyor. Araştrma zamanı:
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/a57972ef-d11d-49f8-895a-2853403615ff)
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/900e4109-8995-4bc5-85cf-0a8e9d033c36)
- Artık ERB ile bir işletim sistemi komutu çalıştırabilir miyiz?
  - Backend’de komut çalıştırabildiğin zaman, sunucuda da komut çalıştırabiliyorsun zaten en büyük etki de böyle oluyor.
- Ruby ERB template injection araştırılıyor.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/05ed2fa4-1629-4a13-aaa2-90c837fcaa3c)
- Pek çok payload kodu var işte.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/60511fbd-6067-4f3c-9c0a-7f9c5c4f31dd)
- Sistem komutu da çalıştırılabiliyormuş. Burdan sonra reverse shell’e gidersin.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/80c34e37-0443-44e0-b8d8-f80a016ae9c2)
## Lab: Basic server-side template injection (code context)
- Kullanıcı girişi yapıyorsun, my account tarafında.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/6c178923-6616-4598-91b4-a81a5c948489)
- Ayarlarda kullanıcının hangi ismiyle forumda gözükeceğeine karar veriyorsun.
- Tonardo engine {{}} şeklinde kullanıyor diye öyle yazdı ama adam zaten onun içerisine koyuyor parametreyi o yüzden gerek yok.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/0b6bdeca-53a9-47ea-9865-3041215dbf1b)
- object yazınca objeyi direkt bastı yoruma. Objeyi enumerate edebilir
- python komutları denedi, code evaluation var diyor.
- asfasjhd yazınca da hata aldı buna da normal dedi çünkü string bir şey oluşturuyor, sonra gitti kaynak koda baktı.
- import os yapmaya çalışıyor, \n falan denedi, {%import os%}
  - importun farklı bir yöntemini bulmak lazım.
- doğrudan bir python kodu yazmak lazım.
- en son şunu yazdı: __import__(”os”)
  - farklı bir import şekli
  - __import__(”os”).system(”rm -rf
  - /home/carlos/morale.txt”)
- Bu template engine, oluşacak template’i satır satır ayırıyor ve aldığı inputu string olarak evaluate ediyor ama sandbox içinde çalışmıyor sorun orda.
## Lab: Server-side template injection using documentation
- Rich-text’i kullanıcıdan alıyor, bu uygulama da java.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/44c9ea59-5ddd-48a9-9c5d-c310842d1816)
- Öncelikle ${Object} deniyor hepsinde. Bu örnekte FreeMarker template engine varmış.
- illa gidip ${asfsa} bu şekilde devam etmen gerekmiyor. Bütün taglere, macrolara falan erişim var.
- İnternetten bu template injectionu için döküman inceliyor.
- Kendi localinde böyle bir ortam oluşturmak lazım, break point markerlar ile dökümantasyondan inceleyerek ilerlersin. Bir sınıf nasıl initiate(oluşturulur) edilir ona bakmak lazım. Bu iş öyle ilerliyor.
- Exec class’ını oluşturmayı öğrenmek lazım sonra exec fonksiyonlarını kullanarak çözecek.
- Backend’de objectconstructor set edilmiş. Sonra assign edilmiş bu obje.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/2680bdc0-e7e6-45c9-a017-0fcce35edc6d)
- Google’dan da bakıp çözülebilir ama bakış açısını göstermek için sağ sol yapıyor. Class oluşturma yöntemlerine bakıyordu.
- Bir rapordan bunu buldu. Execute adında bir sınıf var(full ismini yazmışlar oraya) yeni bir sınıf oluşturuluyor ondan. İşletim sisteminde komut execute ediyor bu sınıf.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/4f9310cd-b3f9-4165-a46c-a5c54a9642e2)
---
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/223f6475-07d3-450b-92aa-2a959cc2b863)
Gelen eposta içeriğini Server side render etmiş Uber. Tabiki render edecek ki oluşan içeriği mailin içerisine yazsın.
---
## Lab: Server-side template injection with a custom exploit
- Giriş yapıldı, avatar gelmiş bu sefer.
- Resim yükledikten sonra kaynak koda baktı bu resim nerden geliyor gibisinden.
- Böyle bir path’den çekiyormuş avatarı. Weiner’ı değiştirse orada bir template injection olabilir.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/7430fa80-b70c-4f06-9efe-9bacbfb54e9e)
- Bir yere gidip yorum yaptı ve orada da kaynak koda baktı.
- Change email kısmında template injection denedi.
- Avatar yüklediği paketi inceliyor. Resim olarak göndermeyeyim dedi sonra username’i bizden aldığını gördü şunu denedi ve “Unauthorized” hatası geldi.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/60b25a61-7d5a-4b16-b840-99220629cecf)
- Resim yerine asfasfas yazdı ve istek gitti. Resim isteğini çağırınca asfasjagsjd{7*7} falan geldi çok saçma…
- resim yükleme paketinde Content-Type’ı text/html yaptı. Önceden image/jpeg di ve response olarak da text/html döndüğünü görünce kafası karıştı sonra resim paketine bakınca kafası yerine geldi.
- text/html yazınca php kodu geldi oraya evalution yapmaya çalışıyor. Şunu denedi ama başaramadı.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/ea30aabc-69a5-4c4a-8239-b54cc1b823b7)
- Bir resmi gidip o klasörün altına kaydetme fonksiyonuymuş bu ve gidip tekrardan çağırtıyor sonra image olarak kaydettiriyor.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/087986c3-b46b-4c7b-b1bb-caf8d46623e5)
- Sonra resim isteğini tekrar yollayınca asfasjagsjd{7*7} geliyor olmadı yani 🙂
- Yine bir okumama vakası…
- setAvatar’ı çağırmamıza gerek yok
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/885c487d-8d27-408a-b8fc-f147be9fe037)
- En son çözüme baktı :(
