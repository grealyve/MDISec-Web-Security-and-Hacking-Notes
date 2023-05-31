<h1 align="center">Source-Code Review</h1>

#### eval her zaman sıkıntı bir fonksiyondur, bu fonksiyonun parametresinde kullanıcının kontrol edebildiği hiçbir şey olmamalıdır.
# Kod Analizi
## Command Injection
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/7af9f79f-d8fb-4f8b-aefb-0266ba8969ac)
- .Net yazılımı bu, IntallPackage dediğinde aklına işletim sistemi üzerinde bir paket kurulumu olacak bu da arkada background proccess oluşturacak, directorylere bir şeyler yazılacak.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/8d9b0660-f44a-49f4-a2f9-6baba300f38d)
- Burada Command Injection zafiyeti var. Process’in Argüman’ına string concatenation yapılıyor Argüman passing yapılmamış durumda.
## Code evalution
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/961a1e06-ebcd-4031-8801-4708624e0ac8)
- Bu input alanı genelde HTML oluyor. HTML templating var fonksiyonun ismi o zaten.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/27863e50-28d9-46c9-a293-d7453732eb37)
- Code evalution zafiyeti var burda. Eval fonk. parametresini son kullanıcı değiştirebilir burda oynamalar yapabilir.
## SQL injection
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/82a4ba9f-4271-4378-b94e-8d6f752edb33)
- Bu kısım kıllandırıyormuş:
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/0e471a08-0f61-467e-92f4-6c94eb145ad1)
- “nodeParm” kodu genel olarak incelediğinde, GET ve POST requestleri ile birlikte gelen ve userın kontrol edebildiği bir parametre.
- Java kodu olduğu için, Java’da sqlRestriction kodunu gidip okuduğunda SQLi’dan kaçınman için gereken bir fonksiyon. “sqlRestriction” fonksiyonu virgülden sonraki nodeParmValue gibi parametreleri kontrol eder. Fakat burada “nodeParameterName” query template’dir ve kullanıcı manipüle edebildiği için SQL injection çıkar.
### NOT: if else kısımlarını okumazmış, neyi trace edeceğine bakar eğer o parametre if else te geçiyorsa bakarmış.
## KeySize
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/22d1088d-ef1e-4b48-b8c5-4f8370ec5a04)
- Burada yönetilebilen parametre tokendır. Direkt token’a odaklandır reis. Token’ı manipüle edemiyorsa okumazmış.
- KeySize sen gidip 4096 yazdığında gidip değiştirmezmiş. Setter ve getter lar ile değiştirmek lazım diyor.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/ab7bc6e7-45b9-4ac8-9462-af20d33a85e0)
## Local File Inclusion(LFI)
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/6aeb20aa-8ebe-4ba3-8809-af3d6ca9b3b8)
- Regex gördüğün anda şüphe uyandırır.
- require_once dediğin anda burada Local File Inclusion vaar.
- cookie ile bir şeyler geliyor. Regexi kontrol ettiğinde 46-122 arası karakterleri yasaklamış ama işte $ ile başlayan bir şey koyarak yine LFI yapılabilirmiş.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/210e5abf-0618-48f4-a0c3-9001fb7e02b3)
```sh
$../../../../../../../../etc
```
## XXE
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/494ec36e-d8fd-4b8d-a8b4-0273675a894d)
- XXE varmış burda
    - .odt dosyası
- SAXBuilder ile aşağıda dox.getContent çalışınca, content.xml ‘i parse ediyor ve DTD lerin disable edilmesi yokmuş.
## Log Forging Vulnerability
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/3e0ed137-d43f-4cda-8673-49af5ebf3142)
- _create_export bir file oluşturuyo, bid ne işe yarıyor ona  bir bakmak lazım hem input alanı zaten. Bu fonksiyonlar ne iş yapıyor bilemediği için gitti üsttekilere baktı
- business_id ile log injection yapabiliyorum dedi. Devamını görmek lazım işte…
- Jsonify outputu content type olarak json verirsen bile, body’i kontrol edebilsen bile browser render ettirtmiyor dedi
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/6c8cd6aa-165a-4b98-9078-77949fe36120)
- Line 22’dee geçici log dosyası oluşturuluyor ardından siliniyor, buna injection yapılabilir. Bunu da söyledi
## Deserialization
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/a94f9101-6a14-4bfd-83ee-a24032dd2b26)
- Untrusted source’dan aldığın datayı deserialize ediyosan .Net, Java gibi OOP dillerinde ciddi problem.
- String’den direkt objeye dönme işi var burda.
- “textReader” Deserialize ediliyor, bu da zaten input alanında var ayrıca stringi XML ‘e çeviriyor.
## XSS
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/ebaa31f2-e386-4c12-b996-030d92ac97e2)
- $upload_name kısmında XSS varmış.
- 21. satır upload_name kullanıcıdan alınıyor ama sanitize ediliyor fakat sanitize etmenin hiçbir önemi yok. Burada HTML encoding yapman lazım.
- 13. satırda echo veriyor burda da XSS varmış
- Ayrıca 19..satırda upload_name kısmında hiçbir validation yok, burda da File Upload vuln. var.
## Arbitrary File Overwrite
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/cd89c1c4-12f6-461e-bdbb-aad88e36ca52)
- installRepository diyor, bir şeyler yüklenecek.
- Arbitrary file overwriting
- mode’u benden alıyor, repHome’u benden alıyor, repHome’u istallConfig’e yolluyor yani “dest” kontrol ediyoruz. Gidip config dosyası üstüne başka dosya yazır falan yani
## Regular expression Denial of Service(ReDoS)
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/c247e391-e4b7-481b-95c7-9135f207d964)
- ReDoS var burda.
- 36.satırda Regex(search) var.
- Non-Deterministic Finite Automaton
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/dc6e3348-1f96-4eb3-8d3d-3c7434fe38a8)
- Recursive inner call’a girer bu da DOS’a sebep olur. a ile başlıyor b ile bitmesi gerekiyor, aa ile başlayıp a ile devam ediyor b ile bitiyor…..
- .Net ile yazılıp NFA kullanan motorlarda bu sebep olabilir.
## LDAP Injection
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/fda5ae39-2118-4d1a-ba27-d278d2ff78a6)
- LDAP search injection.
- 28.satırda ldapSearchFiler alıyor, filterın içine atyıor, 33te de ldap_seatch ediyor işte. 30.satıda req ile envvar alıyor.
## ZIP Slip
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/c130727e-c2da-47a8-9571-3c981154da25)
- Zip’i decomprise ederken programlama tarafında, çok dikkatli olunmalı. Zip’in içerisindeki syn-linkler ile dosyayı çıkardığında *path traversal check* yapmak lazım
- Yani içerden ../../../ dosyası falan çıkarsa ne bok yiyecen??
## Arbitrary File Deletion
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/51c4268e-abe7-418b-9a50-bb3e8473475b)
- get_addon_path adından belliymiş
- gidip token alınıyor, temporarily bir path oluşuyor, bu dosya geçerli mi diye bakılıyor, sonra gidip o siliniyor.
- Arbitrary file deletion.
- tmp_token kullaınıcıdan alınıyor ve artık pathi o kontrol ediyor falan…
## XSS
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/d7226085-5322-447c-a4b3-d96585941d87)
- XSS var.
- Html.Raw, Razor template engineinde default context encoding yapmaz. Default contex encodingi HTML ve attribute encoding için yapıyor.
##  Command Injection
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/96ac19f0-484a-470c-aa93-7a8eb7ce72f0)
- Şu patterni gördüğün zaman renkli hikayeler dönüyor demektir. File’a “name” attribute’u atanması lazım burda da 8.satırda bütün alanlarına koymuş zaten
## CORS Bypass
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/b4379195-1af8-4f18-aa13-60c41dea7f0c)
- Cors bypass var:
  - referer header’ı parametre olarak alınıyor
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/67aef83c-3045-4ba9-a05e-98beea9430fe)
- 14.satırda referer değil origin header’ı tarafından check edilmesi lazımmış CORS konusu.
- Origini kontrol edip spoof edemiyoruz fakat referer’ı kontrol edebiliyoruz
## Account Takeover
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/772f1afc-3a9d-4bc2-9c74-06588dda891a)
- Account takeover…
  - password reset veya acount oluşturma olayı var  bruda
- Sana doğrualam epostası geliyor ya, bu token’ı offline olarak hesaplarsam sıkıntı
- tokenı oluşturma fonksiyonu tahmin edilebilir.
- CSPRNG
## Path Traversal
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/f06f88c0-3219-4eab-98fa-90a41c0546ff)
- \ ile de bakman lazım…
- image’dan LFI yaparsın akarsın
## XPath Injection
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/00414b51-7270-456d-995d-e7540dabd300)
- Kullaınıcdan alınan inputu direkt olmasa da query kısmına yapıştırıyor.
- XPath injection
## SSRF
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/3e7c914e-af7f-42ee-bd28-a44b64b17473)
## Open Redirect
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/7cd22bb5-af11-4c16-b8b9-b9b3e9f2e996)
- Django kullanılmış.
- get_success_url returnüne redirect verilmiş
- "next"e [google.com](http://google.com) yazarım akarım
