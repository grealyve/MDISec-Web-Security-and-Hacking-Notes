<h1 align="center">XXE Injection</h1>

## XML Nedir?
- Bir datayı export ederken ,structural tuttuğun bir veriyi, başka bir ortama taşırken kullandığın yapı formatı. (Web app olur)Servislerin ve programların veri alışverişinde kullandığı ortak bir dil. Günümüzün JSON’ı gibi.
![1](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/49d850fe-61ba-4216-bc22-809a88584a30)
- Uygulamaya dış dünyadan gelen veriler “*input*” ise başka bir web app.den gelen data da “input” tur.
    - XML’i aldığında parse etmesi lazım. Input validation için de parse etmesi lazım.
    - Bizim hedeflediğimiz kısım ise programlama dilinin parsing yaptığı kısım ve an.
- Bir uygulama XML input alıyorsa her zaman XXE’ye dokunur.
- Adam XML verisini SQL sorgusunda kullanıyor ise, SQLi aramaya devam et.
![2](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/b526fa7b-30ee-4ebb-949c-347404a92715)
- XML’i parse edip DB’ye kaydedip, o verileri sana başka bir ekranda sunuyor ise burada Stored XSS bakabilirsin.
- XML için class oluşturur gibi DOCUMENT type oluşturuyorsun:
- DATABASEin tablosuna göre bir formatta yapı oluşturmuş oluyorsun.
```xml
<!DOCTYPE note
[
<!ELEMENT note (to,from,heading,body)>
<!ELEMENT to (#PCDATA)>
<!ELEMENT from (#PCDATA)>
<!ELEMENT heading (#PCDATA)>
<!ELEMENT body (#PCDATA)>
]>
```
Bu dökümanın tipi : note
Bu notun elementleri : to,from,heading,body
Bu elementlerin tipleri de str, int gibi şeyler olacak yanlarında yazıyor.

- Bu oluşturduğun dökümanı XML structure içine import edersen 
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE note SYSTEM "Note.dtd"> --> Bu kısım import etmek (External Document Type Definition)
<note>
<to>Tove</to>
<from>Jani</from>
<heading>Reminder</heading>
<body>Don't forget me this weekend!</body>
</note>
```
- Bunu yapmak sana doğrudan bir "note" dökümanı(classı) oluşturabilmeni sağlıyor.
- <!DOCTYPE note SYSTEM "Note.dtd"> kısmında "Note.dtd" dosyasını gidip okuyor.
  - Yani bu demek oluyorki local sistem dosyalarından birini okutma kapasitesine sahibim.
- Bu adama string ifade vererek de XML document oluşturabilirsin sıkıntı olmaz:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE note [<!ELEMENT note (to,from,heading,body)]> 
```
- DTD içine "Entity" yazma olanağı sağlar. String ifadeler yazabilirsin yani.
```xml
<!ENTITY writer "Donald Duck.">
<!ENTITY copyright "Copyright W3Schools.">
XML example:
<author>&writer;&copyright;</author>

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE note [<!ENTITY writer "Donald Duck."]> 
<note>
<to>&writer;</to> --> Donald Duck yazacak.
<from>Jani</from>
<heading>Reminder</heading>
<body>Don't forget me this weekend!</body>
</note>
```
- Entity nin güzel özelliklerinden biri SYSTEM operandına erişme özelliği barındırması.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE note [<!ENTITY writer SYSTEM "http://x.com/"]> yaptırtabiliyorsun. 
```
#### Web application senden XML requestini aldıktan ne yapıyor çokomelli!!! SQL sorgusu mu yapıyor başka bir şey mi yapıyor??

- Syntaxı bozarak error vermesini sağla, erroru geri dnüyor ise istediğin içeriği bu error’un içnide getirt. Error based SQLi gibi.
![3](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/e8d1a190-c86a-467c-be70-06af61819e07)
< Bu şekilde hata verecektir
- 2 yazan yere askfaskhjfasjk yazsan bile hatada sana bunu döndürdüğü için hackleyebildin.

- <!DOCTYPE note [<!ENTITY writer SYSTEM "file:///etc/passwd"]>
![4](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/27b87dd9-86a7-44a6-9606-604e1c0f9c21)
- <!DOCTYPE note [<!ENTITY writer SYSTEM "http://127.0.0.1:9002/"]>
    - Sunucu üstünde çalışan Elastic Searche http isteği ürettirebiliyorsun. SSRF zaafiyeti ortaya çıkıyor.
- http://169.254.169.254/latest/metadata  —> EC2 lar bu URL’e erişim imkanına sahip. Burada EC2’nun kritik bilgileri yatıyor. AWS kendi otomatik cevap dönüyor buradan.
- <!DOCTYPE note [<!ENTITY writer SYSTEM "http://127.0.0.1:9002/"]>
    - Sunucu üstünde çalışan Elastic Searche http isteği ürettirebiliyorsun. SSRF zaafiyeti ortaya çıkıyor.
- http://169.254.169.254/latest/metadata  —> EC2 lar bu URL’e erişim imkanına sahip. Burada EC2’nun kritik bilgileri yatıyor. AWS kendi otomatik cevap dönüyor buradan.
![5](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/5a24666d-fd50-4be4-bdf6-96f17ce81506)
- x.com’a gitse istek atsa ve oradan gelen cevap da XML olsa ve Web app. bunu işlemeye devam etse…
- txt yerine xml dosyası okuttuğunda <> işaretlerini tag olarak algılayacak ve hata verir muhtemelen. Encode özelliği olmadığı için encode da yapamazsın.

## XXE Out of Band
- x.com/test.dtd ’ye istek atıyor. x.com da bu web app.a istek atıyor ama 2 tane entity var.
![6](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/e69856bb-3403-4b47-96b5-041d29022612)
![7](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/4db33b89-c9c7-420d-8621-c9f8d48dbf99)
settings.xml dosyasını “payl” değişkenine koyacak.
alttaki entityde tekarar x.com’a istek gönderecek parametresi de az önceki xlm dosyası olacak.
![8](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/29d1e133-b6a3-4f1f-be90-4c6643f36249)
XXE yi tespit etmek için şu satır yeterli. Burp collabrator client oluşşturdu onun URL’

#### Ekstra Notlar:
- bir web app .docx .xlst alıyorsa senden bunu parse etmek için “unzip” yapması gerekiyor. [Content_Types].xml dosyasına :
<!DOCTYPE foo [<!ENTITY xxe123 SYSTEM "http://127.0.0.1:8000/"]>
ekle sonra ziple tekrardan ve sunucuya yolla
- XLST parsing mevzusu var
