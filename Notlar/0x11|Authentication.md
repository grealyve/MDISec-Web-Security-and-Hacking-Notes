<h1 align="center">Authentication</h1>

### Kısa Bir Authentication Tekrarı
- HTTP request-response cycle’ının en büyük problemi, bir sonraki req-response ile hiçbir bağı(iletişimi) olmamasıdır. Bunun yarattığı en büyük problem ise; her req-response cycle’ı için kullanıcımız kul. adı. parola girmesin diye, authentication tamamlandıktan sonra bu vatandaşı tanımaya devam edebilmesi için Cookie diye bir mekanizma geliştirilmiştir.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/95d842d8-f9f7-4732-b12d-1f9de5760d65)
- Cookie ile web uygulamasına oturum anahtarımı taşıyorum, ki benim kim olduğumu söylesin.
- Oturum anahtarı her zaman authentication ile ilgili bir husustur ve web uygulamalarında authenticationla ilgili ilginç buglar çıkmakta.


## Lab: Username enumeration via different responses
- Bilerek yanlış bir login yaparken eğer aldığın hata “Invalid username” ise DB’de bu username var mı yok mu diye kontrol eder, var ise parolayı da alıp if ile karşılaştırıor.
- Bu yaklaşım yanlış değil. Aldığın username’i, sessiona hangi dataları koyacaksan onları DB’den WHERE clause ile çek, sonra passwordları karşılaştır. Devamında çektiğin dataları da sessiona da yazabilirsin.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/ee56a718-3e59-41b6-9251-1e9cbd6a4b29)
- Yanlış olan, invalid username geldiğinde sadece “username invalid” şeklinde hata dönmesi.
  - Burp içinde Grep - Extract komutu yazdırabilirsin çıktıları daha iyi filtrelemek için:
  - Invalid kısmının geçtiği yeri seçip OK dersen yeterli
- Labı brute force ile çözüyorsun.

- Bu Labdan çıkarılacak ders:
  - Session, daha kullanıcı ilk geldiğinde oluşturulur. Oturum açtıktan sonra Cookie’deki değer regenerate edilir.
- Sessionda bir değişiklik olduğunda CSRF token da rotate edilir. Bazı botlar her req-response’da bu tokenı rotate ediyor.
  - CSRF token Session içerisinde barındırılır. Sessionu yok edersen CSRF de boştur.

## Lab: Broken brute-force protection, IP block
Bu labda brute-force protection bulunmaktadır bu yüzden nereden bu korumayı sağladığını tespit etmemiz gerekiyor öncelikle.
- Hatalı deneme sayısını ölçtüğü yer Session olabilir. Cookie’yi sildikten sonra tekrar login olmayı denersin ve sana yukarıda Set-Cookie ile yeni bir cookie verir.
    - Manuel olarak kaç kere denedikten sonra blockladığını ölçmek isteyebilirsin. Backend tarafında Sessiona özel sayaç tutuyor, belli bir eşik değeri geçince baaaaam diye engelliyor.
    - Burte force yapıcaksak her 3.requestte Session’ı silip öyle denemeliyiz.
- Belki de IP tabanlı bir şey yapıyordur. Hikaye burada uygulama senin IP adresini nasıl alıyor? Nasıl hesaplıyor kısmına geliyor. (https://www.mehmetince.net/yuk-dengeleyiciler-ve-gercek-ip-adresi-karmasasi/)
    - Arka taraftaki reverse proxynin arkasında bulunan web uygulaması, bir takım headerlara bakar. Bu nedenle engeli aşmak için ilk denemen gereken şey:
        - X-Forwarded-For: 127.0.0.1 headerı ile sınırı aştıktan sonra 127.0.0.2 yapıp denersin, banlanırsan g.o
        - X-Real-Ip: 127.0.0.4 üstteki gibi denersin.
            - Param miner extensionu var, response a göre karar veriyor. Guess header
  - Headerları banlandıktan sonra Intruder ile teker teker denedi blocku aşabilmek için. Fakat hiçbiri başarılı olmadı.

- Ameleus yöntemi ile 2 kere yanlış deneme + 1 kere login olma şeklinde ilerlemeyi çözdü. Yani Cookie’yi sıfırlamak gerekiyor her 2 denemede.
- Burp Copy as Python Requests
    - username passwd seçip, sağ tıkla, Copy as request with session object. Python kodunun bir kısmı şu şekilde:
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/83478ee9-4239-4060-8b97-b08c1246da96)
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/05361680-82c8-4e89-b0fc-861d8fecfab0)
freedom
