<h1 align="center">SonarSource 2021 - Güvenlik Zafiyeti Takvimi Challenge !</h1>

## Missconfiguration
- İlk dikkat çeken husus aslında SQL sorgusunda kullanıcıdan alınan bir inputun bulunuyor olması. Burada URI parsing ile alınan bir input var, browserdan denersen URL encoded gidecektir muhtemelen o yüzden brup gibi toollar ile denemek lazım.
- 7. ve 8. satırda
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/7c414a4c-0d64-4728-aa26-bf7b80c37013)
- Buradaki asıl olay “test” fonksiyonunda. İlk case için true, ikincisi için false dönmekte.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/9ec91adf-3ff3-4d52-bc5c-06971c0f6ec9)
## SSRF
- Burada kullanıcıdan inputa istek gönderiyor, ve gelen cevabı da sana geri iletiyor. Proxy gibi ama burada SSRF zafiyeti var ama bir fark var. Web sitesinin hostun yani gideceğin domainin DNS cevabı belirtilen 2 ipden biri olmak zorunda. Köppek gibi whitelisting yapmış burda. 
- Bunun atlatmanın ilk yöntemi şöyle:
1) Gelen domain listede mi diye kontrol edildikten 4 satır sonra o web sitesine istek atılıyor. Yani sen aslında DNS talebinin TTL’ini 1 ya da 0 saniye yaparsın. Adam DNS sorgusu yaparken bunu yollarsın sonrasında hemen istediğin ip adresine değişirsin ve atlatmış olursun. Time to check time to use zafiyeti. 
- inputa https://h.com yazmış ol. Bu domainin DNS sunucusu sende olduğu için bunun ip adresini 10.120.1.2 yapacaksın ve TTL’i 0 yapman lazım. 17.satırdaki kod tekrardan DNS çözmek zorunda olduğu için sen de o sırada IP adresini tekrar değiştirip gitmek istediğin lokasyonu yazacaksın.
  - toc 2 dns server change?
Atlatmanın ikinci yöntemi:
2) WebRequest.Create(url).GetResponse() bu adam 302 Redirectionları takip edip etmediğini bilmiyoruz. Eğer takip ediyor ise bu listedeki IP adreslerinden birisi muhtemelen sunucunun local IP adresi, bunun üzerinde Open Redirect zafiyeti bulmak gerekiyor.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/e9e30a82-2e94-4d2e-a471-f63561300301)
## Directory Traversal
- Adam arka tarafta processBuilder’a parametreler vererek bir komut çalıştırıyor. 
- “tar” komutunun bir parametresi var ve bu da bir işletim sistemi komutu çalıştırabiliyor.
- Burada localdeki “backup” ile tüm dosyaları tar komutuna veriyor. İşin kötü yanı ProcessBuilder’a bash vermesi.
  - Attack vektörü olarak “--checkpoint=1 --checkpoint-action=exec=/bin/sh” dosya adı bu olarak tanımlarsanız, tar komutuna bir parametre enjekte etmiş oluyorsunuz.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/3d70c5b8-eb90-41fc-9306-bef168d5ef55)
## Local Privilage Escalation
- environment olarak modprobe’a PATH’i set ediyor. Doğal olarak bu process çalıştığında modprobe çalışacak. Hiçbir parametresini de kontrol edemiyoruz. 
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/c4f47baa-aedd-4916-bdcd-8c59e352380e)
- Linuxta şöyle bir şey var; her binary çalışırken bulunduğu PATH’in variable larına erişimi vardır. Böylece işletim sistemi, processe parametre atayabilir linuxta. Bunu da terminalde “env” komutunu yazdığında çıkar.
- Yani linuxta bir komutun “env” erişimi varsa, env yazdığında çıkan parametrelere de erişimi var demektir. Parametre olarak bu değerleri alabilir.
- Linuxa “Adress Space Layer Randomization” ilk getirildiğinde sizin “Stack” alanında gitmek istediğiniz adres alanında ASLR olduğu için Linux’un ilk versionlarında bunu sadece “Stack” için getirmişler Path’e getirmeyi unutmuşlar.
- İşletim sistemi bir processi ayağa kaldırdığında parametreleri Stack’e pushlar iken aynı zamanda Path environmen(env)teki bütün parametreleri de Stack içine pushluyor adam ki çalıştırılan komut(binary) bu parametrelere erişebilsin. Doğal olarak Address Space Layer Randomization yani Stack alanındaki o değerler randomize edilirken fonksiyonların “epilogue” ve “prologue“ değerleri arasında yapmışlar. O yüzden saldırganlar shell kodu örneğin : SHELLCODE=selam yazıyor. Bir komut(binary) çalıştırdığında, bu SHELLCODE Stack’e gönderildiğinde adresindeki distence sabit olduğu için bu farkı hesaplayıp SHELLCODE’a erişimi sağlanıyordu.
- Elimizdeki koda bakacak olursak modprobe’un hiçbir parametresini kontrol etmiyoruz ama PATH’den input alan bir komut(binary) ise bu PATH değişkenini override edip(kullanıcı olarak yetkimiz var) sonra bunu root yetkisiyle çalıştırıp Local Privilage Escalation zafiyetini tetiklemiş oluruz. Yani bu komutun dökümantasyonuna baktıntan sonra payloadımız şu şekilde olmalı: MODPROBE_OPTIONS=SELAM . Koddaki modprobe bloğu root yetkisiyle çalışırsa bizim configüre ettiğimiz şekilde yükleyecek.
## Memory Leak
- İşletim sisteminin env. yüklenmiş bir “KEY” i strlcpy() fonksiyonu ile passw içerisine atıyor.
- 16.satırdaki strlen() fonksiyonu 16 Byte data yollandığında null byte yok ise komple memorydeki bilgiyi ifşa edecek. 
- Sen 16 Byte gönderiyorsun 13.satırda adamın null yaptığı yere yazılıyor fakat null byte koyulmazsa strlen() metodu string ifadenin bittiğini göremeyeceği için bu değer 16 dan çok daha büyük olacaktır. Stackte “KEY”i geçtikten sonraki bir yerlere kadar gider, memory leak olur.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/6d0051c2-6017-4294-b586-937b9e99d2df)
## File Upload Raise Condition
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/9f7ee221-0b89-4073-a9c8-81edbf6e1a77)
- 11.satırda yüklenilmeyecek dosya uzantılarını blacklist yapmış. Extension için blacklist yapmak çok yanlış.
- Apache 2.4 versiyonundan sonra .phar uzantısı da PHP interpreterına verilen bir default configürasyonu var. Bu şekilde bu zafiyet atlatılabilir.
- 2.bir yok ise, .htaccess konfigürasyonu yüklersin içeriye ve bu aktif olur. blocklist içinde “.htaccess” uzantısı yasaklı olmadığı için yüklenebilir ve bunun kuralı geçerli olur. Bunun içine de kendi kuralını yazarsın: uzantısı “.mdisec” olan dosyaları PHP interpreterına ver diyebilirsin.
## Local File Inclusion
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/2c2ac79c-c090-46e2-89fc-fc1f63ce5aeb)
- 11.satırdaki os.path.join() fonksiyonu sıkıntılı Python’da. En sondan gelen “filename” root içeriyor ise ilk iki parametreyi iptal ediyor.
- 12.satırda directory traversal yapmana müsade etmiyor. Yoksa yapılırdı yani.
## XSS
- 11.satırda kullanıcıdan gelen dosyanın adını 1:1 aynısını almış $orig ile.
- 12.satırdaki kod, $orig deki “..” ile karşılaştığında true dönüyor.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/8a05d1b6-326e-4f2c-9c96-9f1d90ad36b8)
- 17.satırda upload dosyasının altına random bir şey koymuş. Sonra dosyanın gerçekten bulunduğu yeri almış, filename’i RealPath’e gönderiyor.
- Apache bazlı bazı sıkıntılar var. Bir URL’e HTTP talebi gönderdiğniiz zaman bunun extensionına göre Apache2 çalışır. Yüklenen bir dosyayı geri verecekseniz kullanıcıya, geri serve mi edilecek? yoksa download mı ettirilecek? bu ikisi farklı şeyler. Bu dosya resim ise doğrudan “Response” dönebilirsin, download ise “Content-Disposition” dönersin.
  - Content-Disposition ile response’da ne yazarsa yazsın browser bunu interpret etmez, doğrudan file olarak alır.
- Bu kodda da Response headerında filename kısmında hiç böyle bir şey yapılmadığı için; ben içeriye HTML yükleyebilirim, içeriye HTML yüklediğim için de daha sonrasında yüklediğim HTML serve edilirken browser bu HTML’i çalıştırır. Zira HTML serve edilirkenki halinde Content-Type kontrolü yok. Böyle bir kontrol ne alınırken ne de sunulurken var.
## Unicode Exploit
- Unicode’ları upper yaptığınız zaman en yakın unicode dışındaki karaktere çevriliyor.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/19995302-1ce8-4078-bef7-bbe024fa9bf7)
- Reset passwordlerde ORM kullanıldığı noktalarda sizin başarabileceğiniz en isabetli nokta “Unicode” gibi hikayelerdir. Unicode’a en yakın sıkıntı yaratan dil ise Python dilidir.
- Siteye kayıt olurken “admÏn@x.com” olarak kayıt oluyorsun. Databaseye kaydederken unique oluyorsun. Ama parola sıfırlarken bütün karakterleri büyük yaptığı için “Ï” harfi büyük I ya dönüşüyor ve adminin e-posta ile eşleşiyor. Burada ise başka bir isanın tokenını kendine e-posta attırtıyorsun.
- E-maili databaseden gelen e-mail olarak almıyor, inputtan gelene mail atıyor. Databaseden gelen maile atsaydı hiçbir şey yapılamazdı.
## ?
- Adamın rolünü Session’dan int32 bit olarak alıyor.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/232722f1-9d67-412b-981e-8f9354c0006d)
- Kullanıcının hiç rolü yoksa 1 oluyor. 12.satırda rolümüz 1 iken neye cast ediyor bu 1 değerini buna bakmamız gerekiyor. C# dilinin yapısından kaynaklı enum içerisinde var olmayan bir değer sorgusunda 15.satırda casting yapmıyor. Yani hiçbir zaman 17.satır çalışmayacak ve false dönmeyecek.
  - C# dilinde enum, int olarak geçiyor. Value bazlı olarak çalışıyor enum.
## RFI to RCE
- fileName ve content kullanıcıdan alınıyor.
- 9.satırda content’in geçerli bir XML olup olmadığını kontrol etmemiş adam.
- İkincisi “.xml” ile bitmesini istemiyor ama 60.karakterden sonrasını kesiyor. O yüzden 60 karakter yazıp sonrasına .JSPyazarsın, 11.satırdan bu kod geçer sonra 0-59 karakterine de random_name.xml.jsp yazarsın. XML i override edip RCE elde edersin.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/ac4f705d-866f-4339-807c-71f63598c6eb)
## XXE
- Php versiyonu 8’den düşük ise çalışmıyor zaten.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/3826e581-e305-4e03-a01b-2e53a4fe3f9b)
- Her docx dosyası, Zip dosyası compress edilmiş XML dosyalarından oluşur.
- 16.satırda LIBXM_NOENT” yazdığı için XXE var demektir. Entity substitution enable oluyor. Yani 9.satırda if ile kontrol ettiği yeri mahvetmiş oluyor burada.
