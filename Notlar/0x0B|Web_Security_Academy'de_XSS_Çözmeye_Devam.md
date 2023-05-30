<h1 align="center">Web Security Academy'de XSS Çözmeye Devam</h1>

### Not: Bu video için çok çok az not almışım en kısa sürede güncelleyeceğim.
head, body oluşturmasan bile Browser senin yerine oluşturuyor.

- Kendisi head ekliyor ardından body ekliyor, ben svg tagi gördüm “onload” attribute’u var ama tırnakları yok bunları ben koyam diyor.
- new line karakteri varmış gibi devam ediyor sonra < işareti görüyor ama data yok gidiyor boş bir attribute bu diyor.
- peşinden html benim için bir attribute diyor onu da attribute olarak ekliyor sonra kapatıyor.
- sonra bakıyor html kapanmamış, svg kapatıyor, bodyi kapatıyor, htmli kapatıyor.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/3e7109ad-c6f1-4212-b7b2-0ebef257bed8)
