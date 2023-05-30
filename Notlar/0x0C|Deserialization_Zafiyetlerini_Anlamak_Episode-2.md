<h1 align="center">Deserialization</h1>

#### Deserialization için kullanabileceğiniz bir browser eklentisi: Cookie Quick Manager

- Import ettiği classları da oluşturabilir hale geliyoruz.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/a4cf46d9-f4ec-4929-ae5e-1d908f57230b)
- Yazılım aslında “is_admin” değerini int olarak kullanırken sen diyorsun ki bu bir obje. Permission objesini de alıp başka bir objeye point ettirebilirsin.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/0af79155-8877-415e-ad9e-b07859752964)
- Bir objenin propertie’lerini başka bir sınıfa point ettirmek :  property oriented programming
- İçinde wakeup, destruct bulunduran sınıfları kullanarak mesela cache sınfları dosya içine yazma işini yapar
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/90b52c7d-d925-436e-af37-17054328e972)
- Permission’a da git bunu kullan dersin ve backdoor’u kaparsın.
[No-CMS CodeIgniter Encryption Vulnerability Exploit](https://www.youtube.com/watch?v=YYsisTQcxls)

## Lab: Modifying serialized objects [*Lab Linki*](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-modifying-serialized-objects)
- giriş yaparken gelen datayı Burp ile intercept yapıp gelen sessionı decode edip, kullanıcıyı admin yaptıktan sonra tekrar encode edip
yolladı isteği. Sessionı kopyalayıp amelece bütün isteklere yapıştırıp çözdü olayı.

## Lab: Modifying serialized data types [*Lab Linki*](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-modifying-serialized-data-types)
- https://medium.com/swlh/php-type-juggling-vulnerabilities-3e28c4ed5c09
- Kullanıcıdan gelen access_token ı if karşılaştırmasına sokuyor. If comparasion da loose type check olduğu için phpde, zaafiyet çıkıyor.
- 3 tane = koymazsan string ile int karşılaştırdığında True dönüyor mesela kafayı yiyor php.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/f036a7ce-946e-44aa-a87d-0b76775a35f6)
- access_token’ı 0 yapıyor sonra encode edip yollayınca gerisi geliyor.

#### Kralından okuyabileceğin bazı makaleler:
- https://www.mehmetince.net/codeigniter-based-no-cms-admin-account-hijacking-rce-via-static-encryption-key/
- https://www.mehmetince.net/drupal-coder-zafiyet-analizi-metasploit-modulu-gelistirilmesi/
  - https://github.com/rapid7/metasploit-framework/pull/7115
