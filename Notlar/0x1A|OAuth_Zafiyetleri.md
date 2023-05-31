<h1 align="center">OAuth Zafiyetleri</h1>

#### Client application : X web sitesi

## OAUTH Nedir?
- Başka bir web sitesine bilinen sosyal medya hesaplarıyla giriş yapmadır aslında. Bir kişinin iznini(consent) başka bir web sitesindeki bilgilerine erişim sağlamaktır.
  - consent: provider kullanıcıya ben senin şu şu bilgilerini paylaşıcam diyor. Tamam deyince birtakım bilgiler dönüyor ve arkadaş token ile birlikte X web sitesine gitmesi gerekiyor.
- Bir web sitesinde bir kullanıcıya bir dünya form doldurtmak yerine, facebooktaki bilgilerini kullanarak(artık facebook hangi bilgilerini paylaşmak istiyorsa, ismini cismini epostasını…) kullanıcı oluşturma işlemi sağlar. Eposta onaylatmadır falandır o süreçleri hiç yaşamamak için.
- Gelen kullanıcıyı provider her kim ise ona sektirip, geri sektiğinde sana birtakım bilgiler ile gelmiş oluyor.
- Peki birtakım bilgiler paylaşabiliyorsak ben bu kullanıcı adı, parola falan kendi tarafımda tutmayayım her benim sayfama giriş yapmak istediğinde gitsin facebooktan authentication sağlasın ve geri geldiğinde valide olduğunu bileyim.
  - OAuth kullanma şekillerine göre arka tarafta farklılık gösteriyor. [Flows]

- OAuth’da HTTP yi düşündüğünde birtakım sorunlarla karşılaşacaksın. Bir web sitesiyle kullanıcı konuşuyor ve onu valide etmek için başka siteye yönlendiriyor. Bu web siteleri kendi arasında  bilgi paylaşımı yapamıyor CORS mevzuları girer, iki ayrı tab bunlar zaten. Provider bu adamı doğruladıktan sonra geri *X web sitesine yönlendirmesi gerekiyor(browser’a söylemesi lazım)* ve o paketin içerisine de birtakım  bilgiler yazması gerkeiyor(*code*). Bu kodu(token) X web sitesi gidip provider’a sorduğunda işte bu kişi Joseph Stalin diyebilir. Bu 2FA olabilir başka bir şey olabilir..
  - Bunu cookie bazlı yapamaz, başka bir domain için cookie set etmesi gerekiyor ayrıca cookie’nin doğruluğu ne?
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/b66207ab-634d-4267-94bb-1c848b56b9e1)
- Her çeşit providerdan çıkan token farklıdır.
## Lab: Authentication bypass via OAuth implicit flow
- Logine tıkladığında bir tane endpointine gidiyor web sitesinin.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/fcab05e3-ae52-4655-abfb-a0f34d24035a)
- Scope’da sizin OAuth service provider’dan hangi bilgilere erişim sağlayacağınızın kapsamını belirttiğin alan oluyor.
- redirect_url : Authentication tamamlandığında ben tekrar bu web sitesine geri gelmek istiyorum.
- nonce : CSRF zafiyetinin OAuth flowlarını engellemek için konulmuş kod.
- Gerçek hayatta Redirect edip  başka bir web sitesine götürmeli.
  - !!! Bu isteği birden fazla gönderince interaction tokenını (response tarafında) değiştiriyor mu??
- Gidip login olduk.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/2148d7a9-a720-42da-93be-71e5a251e6c4)
- Kullanıcı adı parola doğruysa geldiğimiz siteye geri yönlendirmeli. Başlangıçta ürettiğim kodla redirection yaşadık
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/af86c029-a9c3-4510-b328-df68ab0d6213)
- Provider, senin şu bilgilerini paylaşıcam diye bilgilendirme yapıyor. OK dedik
- En sonda callback parametresine götürdü bizi. Cookie ile session oluşturmuş. OPTIONS ile değerler var. me parametresine gitti.
- Veee arada /authenticate diye POST requesti çıktı karşımıza.
  - provider kullanıcıya ben senin şu şu bilgilerini paylaşıcam diyor. Tamam deyince birtakım bilgiler dönüyor ve arkadaş token ile birlikte X web sitesine gitmesi gerekiyor.
  - Ama bu işlem, vatandaş gidip username passwd ve token gibi değerlerini alıp provider’dan alıp, tekrar X web sitesine POST req. olarak yollarsa işte o requesti tutabiliyor
  - Bu tasarımda token, adam gidip tekrar provider’dan bilgilerini almak için kullanılıyor.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/90997d2f-0f00-46e7-9869-edb70e533d00)
- Ameleliği vatandaşa yüklüyor ama spoof da edilebiliyor bu iş.
- Burda da e maili değiştir.
## Lab: Forced OAuth profile linking
- “nonce” gibi CSRF engelleyici token yok ise, ben gidip birine OAuth linki verdiğimde ya da bu linke yönlendiren bir web sitesine çekebilirsem ve sen gidersen sonra da authentication tamamlanıp gerisin geri redirection callback URL’in neyse o web sitesine gideceksin.
- Web sitesinin hangi bilgileri isteyeceği scope’da belli oluyor.
- client_id : Provider hangi web sitesi için auth sağladığını bilmek için.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/4e8a8c6c-6b04-4f5c-be2b-497fa06ee17c)
- Provider kısmına gidip login olunuyor. Halihazırda login olmuş olunsaydı belki kul. adı parola sormayabilirdi.
  - Ters tıklayıp responselara da bakıyor.
- Burda bir kod ile redirect ediliyoruz. Bu kod 1 kere mi kullanılıyor acep?
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/76d19b95-5043-432b-ba4d-7dfe4c062bb0)
- Bu kodla ana uygulamaya geri gelmedik. Bu istek ile orjinal siteye geri dönüş yapılıyor. CSRF token da yok.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/f346b3cc-61fb-46c8-a2e9-c0a0e1f1453e)
- Bu URL’i admine bu sosyal medya hesabını bağlıyacağız. Bu istekle birlikte bizim sosyal medya hesabımız adminin hesaba  bağlanacak. Sonra biz de sosyal medya hesabımızla siteye giriş yapıcaz.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/204be2ad-0425-4aef-ba68-130d8ca17e4b)
- MDI reisin social medya kodu buymuş hacı diyor amaa CSRF token yok buradaa
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/40396cf8-03b7-4059-96f6-e11c7abbb8c1)
## Burası Çokomelli!!
- Burada başka zafiyetler de çıkabilir.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/412f9fec-cf25-4d12-b0ff-a8b75ed35508)
- Client_id niye var? Çünkü bir providerdan OAuth servisi almak istediğinde adam sana hangi adrese geri redirect edileceğini veya trusted-domain listesi gibi özellikler ister ki bu id ile gittiğinde provider kayıtlarından bakıp redirect edilebilir izinli domainler görebilsin ve seni geri yönlendirebilsin.
  - Birden fazla domainde OAuth servisi kullanmak isteyebilirsin. O yüzden bu redirect_uri üstünde validationlar yapılır. Validationı atlatırsan ne olacak? Validationı atlatamadın, gittin bu domain üstünde open-redirect zafiyeti keşfettin ve kendi web sitene redirect ettirdin ve adamın kodunu çaldın.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/328b6c23-6d85-4147-ba38-b0c57bf391e4)
## Lab: OAuth account hijacking via redirect_uri
- Eğer Return URL üstünde itlik kopukluk yapacaksak ilk istek üstünde durulmalı
- redirect_uri kısmına istediğini yazıp isteği yolladığında sana hala geçici bir token vermeye çalışıyor adam. Yani burada hem senden alıyor URI ve validation yapmıyor bunun üstünde.
- collaborator URLi verdi oraya
- Sonra o HTTP requestini gitti admine tıklattırmak için exploit servera yazdı.
- Sonra sosyal medya ile giriş yaptı ve en son paketi yakalayarak Collaboratora gelen çalıntı istek kodu ile isteği iletti ve admin oldu.
