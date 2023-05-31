<h1 align="center">Burp DOM Invader</h1>

- XSS dünyası browser ve JS tarafında dönüyor. En önemli konu DOM Based XSS. Browser HTML içeriği alıp bir DOM tree oluşturduktan sonra özellikle client side JS implementasyonu manipülasyonları gerçekleştiriliebiliyor. Eğer bu DOM’un içerisinde JS’in kullandığı birtakım metodların parametrelerine müdahale etme imkanımız varsa bu DOM updati sürecinde bir XSS zafiyeti çıkartabiliyoruz.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/3639a42b-82b6-46e0-94a1-d6eafccba116)
- Bir sayfada binlerce JS çalışmakta ve belli durum/yapılarda DOM’u güncellemekte. Bunu track etmek çok zor çünkü içeride binlerce JS çalışıyor.
- Source’dan gelen datalar Sink’e gidiyor ve bunlar nereye gidiyor bunu insan gözüyle takip etmesi çok zor.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/ee94810e-680a-4574-8c7f-36ba89ef2315)
- input alanlarına token belirleyecez, DOM oluşunca bu token nerelerde tetiklendiyse[Hangi JS kodlarını tetiklediyse] oralarda gözükecek.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/c0892769-0ecc-4726-84d8-31228dc2a72a)
## DOM Based XSS
- “trigger” fonksiyonuna gelen parametre “eval” fonksiyonuna gidiyor. Burası sink oluyor.
- Source ise “document.cookie”
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/5a6a10f4-c164-47cc-b5f3-4ceae619e56e)
