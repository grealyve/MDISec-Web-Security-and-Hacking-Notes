<h1 align="center">HTTP Request Smuggling</h1>

## HTTP Request Smuggling ve HTTP/2 Downgrade Attack Zafiyeti Nedir?
- HTTP desync attack olarak geçiyor ****HTTP Request Smuggling. Buradaki zafiyet burda direkt Web ile ilgili(web app. ile değil).****
  - Webin bir tanımı var ve mezhepler bunu farklı tanımlıyor.
- Normal bir akış şeması budur.
  - Load balancer : F5, Amazort Elastic LB, Cloudflare, Cloudfront. 80,443 portunda çalışan bir kod yazılımı.
  - Ngnix de aynı şekilde bir kod ve http’nin bu ikisinde de tanımı aynı. Fakat yorumlama şekilleri farklı bunların. Öyle bir şekilde hazırlıyorsun ki isteği, LB X anlıyor Nginx Y anlıyor.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/f31cacb0-3a4f-43a7-a6e9-f28bfb26947d)
- Three way handshake tamamlandıktan sonra bu TCP oturumu içerisinde HTTP isteği göndereceksin. Aşağıda da bir sürü tcp isteği gidip duruyor tüm paketler gidene kadar da karşı taraf bekleyecek. Tüm gelen dataları karşı taraf yazdıktan sonra ve 2 tane new line gördüğü zaman artık TCP’yi transportation katmanı olarak kullanıp yukarı çıkıyorsun ve 1 adet HTTP requesti göndermeyi başarıyorsun.
  - Karşıdaki HTTP protokolünü anlayan kütüphane  bu serviste işlerini yaptıktan sonra sana cevap dönüyor. Sen bu cevabı alana kadar 2. bir isteği bu TCP sessionu üstünden gönderemezsin. TCP 1’de böyle.
  - HTTP bir text transper protokolüdür. Doğal olarak text gitmesi gerekiyor.
  - HTTPS de ise SSL işin içine giriyor, paket şifreli halde gidiyor ve karşı tarafta decrypt edildikten sonra cevap geliyor falan akış aynı yeni.
- Bu datada http kütüphanesinin dikkat ettiği 2 tane header var. 1-Content Lenght, 2-Transfer Encoding. Bu headerlardaki değere göre LB’da HTTP sunucusu davranışı gerçekleştirir.
  - HTTP’nin tanımında çok net bir kural yok. Content Lenght varsa transfer encodingi kale alacak mıyım almayacak mıyım? İkisi bir arada varsa nasıl olacak? Hal böyle olunca LB’da çalışan HTTP servisini implemente eden adam kendi kurallarını üretiyor, arkada çalışan nginx de kendi kurallarını üretiyor. Bu kurallar arasında da farklılık olursa sıçanzi.
      - Mesela LB content Lenght’e göre bir ayırım yapıyorsa altta kalan kısmı yeni bir HTTP isteği olarak farz ediyor. Ama arkada Nginx farklı bir yorumlama yapıyor buna.
      - ![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/d0c8b0e7-ce8a-47b4-92fc-302be28abe95)
### HTTP 1.0/1.1/2 her zaman alt katmanda TCP protokolünü kullanırlar. 
### HTTP 1.0/1.1 için en kritik şey, sen günün sonunda bir TCP oturumu elde ediyorsun ve bu hat üstünden requesti gönderiyorsun.
### Buradaki asıl mesele zaten bu HTTP paketini ilk karşılayan servisin nasıl bu kuralları implemente ettiğidir.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/f648ccca-c4ae-4407-9683-291edeb28e50)
- Başka bir request injection yapmış oluyorsun. 
- LB bir dünya insanın requestini alıyor ve burda konu diğer insanların responselarını alma veya onların responselarına müdahale etmeye kadar gidebiliyor.
  - Nasıl fixlenir? Önde ve arkadaki servislerin kural tanımları aynı olması gerekiyor. Ya da vendor araştırması yapıyorsun. Aynı kişiden bu hizmetleri alacaksın.
- En temel nokta HTTP 1.0 ve 1.1 için “Transfer Encoding” ve “Content-Lenght” headerlarının protokol için bir anlamı var.

- Ama HTTP 2.0 da bunun hiçbir önemi yok. Böyle bir header yok. Artık binary bir protokol ile karşı karşıyayız. 
- İsteklerin içine id yerleştiriyorlar artık ve adam da bu id değerine göre cevap üretiyor. Yani tek bir iletişim kanalın yok istediğin kadar istek atabiliyorsun. Asenkronizasyon kazandırıyor.
- Binary bir protokol olduğu için requestin nerde bittiğini kontrol etmek için Content-Lenght e bakmana gerek yok. Protokollerin alt katmanında Frame’ler var, frameler içerisinde bitwise kaç bite olacağı yazıyor. Bunun da çok katı bir şekilde validationı yapılması gerektiği söyleniyor. Bu bahsedilen güvenlik açığı böyle bir güvenlik açığı doğrudan yok.
---
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/42366d11-596f-412d-a31a-15b542cc721b)
- HTTP/2: Ttrsequd is Always emeLJarnes Kettle (altimwax)
- Bu adamın farklı bir yaklaşımı varmış :
  - Load balancer HTTP 2.0 ı destekliyorsa, arka taraftaki adama hangi HTTP protokol versiyonuyla yollayacak? Fark etmez.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/9775d334-36ed-41f1-b2dc-a6ae8677d625)
  - Bu da developer açısından şöyle bir sorunu getiriyor: 2.0 ı 1.1 e convert edecek ve arkadaki servis hayatına devam edecek.
  - Ön taraf için hile hurda yapamıyor olabiliriz ama öyle bir HTTP 2.0 paketi yollıcaz ki bu requesti alan yazılım 1.1e döndürdüğü zaman KAFASINDAN duman çıksın.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/a5b20c7c-796b-4fc1-8b3f-0a55efdc7471)
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/8b9280df-9feb-4197-8c8b-1e2c576f23c4)
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/4b5424b5-e69e-49b5-affb-17e338fe6328)
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/23652d01-3237-4e1c-bc28-9a1b8f5d1f94)
  - Load balancer ve Web server arasındaki TCP sessionını nasıl yaşattığına bakıyor her şey.
    - Her bir isteği sıfırdan bir TCP bağlantısı kurarak oluşturuyorsa zafiyetlerin hiçbiri olmuyor.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/82636205-6ba4-49a7-8d0f-1dbf8bf8b6bf)
