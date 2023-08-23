<h1 align="center">XSS Güvenlik Zafiyeti Part - 2</h1>

Buradan [*XSS Çalışma Tahtası*](https://public-firing-range.appspot.com/) XSS talimlerinizi gerçekleştirebilirsiniz.

## XSS Nedir?
- User input’unun developer’ın JS kodu tarafından kullanılması.
- XSS’in nerede oluştuğunu anlaman çok önemli. innerHTML kodu gidip oraya yerleştiriyor ve Browser bunu tekrar parse ediyor böyle olunca da XSS ortaya çıkıyor. Yani username kısmına```<svg onload=alert(1)>```  yazdığında olay olmuyor.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/58919bf2-66fb-4aaf-a013-edc63e9e0913)
1. Kullanıcı gidip uygulamaya üyelik açıyor, username veya epostasını kendi dilediği gibi seçiyor. Bu bilgiler Databaseye kaydedildikten sonra başka bir ekrandaki JS kodu AJAX sorgusu[getUsername();] ile bu bilgileri alıyor.
2. documenet.getElementById fonksiyonu ise bunu alıp DOM’u manipüle ediyor. DOM’un update’ini gerçekleştiriyor. Buradaki hikaye; dış dünyadan kontrol edilebilen bir değişken ile, halihazırda oluşmuş bir DOM’un belli senaryolara göre çalışan Javascript kodunun DOM’u güncelleyip değişkenimizin XSS açığı oluşturmasıyla başlıyor. Yani Browserımız gidip direkt hedef alana payloadımızı yerleştirmiyor, bir fonksiyon gidip bu DOM’u güncelleyince payloadımız yerleşmiş oluyor. 
3. Fonksiyonumuz şunu yapıyor: ID’si “msgArea” olan tagin içerisindeki HTML değer bu olsun.
4. Sen Browsera bunu verdiğin an itibariyle Browser birdaha HTML parsing yapacak ve XSS’e bu şekilde erişmiş oluyorsun.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/9b94408e-dd81-4b71-a634-f3f565f96dbc)
- Senin Javascript kodunu okuyup, hangi fonksiyon nereyi update ediyor bunu anlaman gerekiyor. Son derece önemli!

- DOM’u hangi JS fonksiyonları update ediyor bunları bilmek gerekli. Insecure jquery functions:
```javascript
.html()
.append*()
.insert*()
.prepend*()
.wrap*()
.before()
.after()
```
- window.postMessage() :
    - Bir web sitesi başka bir window’u iframe içinde açarsa bu iki iframe’in güvenli bir şekilde iletişim kurmasını sağlayan hikaye. Çok sık kullanılır. (gmaildeki açılan e-postalar)
    
- Bunu sömürmek için başka bir web sitesi kurup, bu web sitesini iframe ile çağıracak. JS ile kendisini iframe ile açan adamın gönderdiği cross mesajları dinleyip aksiyon alan bir JS kodu var aşşağıda.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/fdb0ac0f-b5f2-4b6a-bbd0-ca3335e6c891)
- Yeni web sitesinden bu iframe içerisine POST message ile bir Json göndericez. Bunu aldıktan sonra Json’ı parse ediyor, parse ettiğinin içindeki datayı html kısmını alıp innerHTML olarak yeni bir div açıyor. Div’in içeriğine senden aldığı datayı yazıyor.
  - innerHTML için tehlikeli kısım, window.addEventListener gelen mesajı postMessageHandler fonksiyonuna yönlendiriyor…
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/b41a75bf-e08d-4f74-86aa-eb777f06ed72)
- JS bu innerHTML’i alıp DOM’un içine ekliyor.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/093651a9-7e37-4290-8a5d-71706038dccc)
- Böyle bir web sitesi oluşturuyoruz.  Önce iframe yüklenmesini bekliyor, yüklendikten sonra “postMessage” fonksiyonu ile iframe içine Json payloadı yolluyoruz. 
- Senin browser bütün cookie’leri isteğin içine de ekliyor.
```javascript
var payload = {’html’ : ‘<svg onload=alert(1)>’}

var payload = {’html’ : ‘<img src=x onerror=alert(1)>’}
```
bu payload karşıdaki(iframe içindeki) websitesinde çalışır bunu da şu şekilde anlarsın:
```
var payload = {’html’ : ‘<img src=x onerror=alert(document.domain)>’}
```
Bu websitesinin asıl amacı kendi iframeleri arasında iletişim sağlamak. Başka bir iframe içinden gelen istekleri kontrol etmeleri gerekirdi. postMessage()’ların güvenli kaynaktan gelmesi ve innerHTML yerine de sayfanın tüm içeriğini yeniden oraya vermektense(browser’a tekrar yorumlatıyor) data() metodu ile sayfanın datasını değiştirmesi daha iyi olurdu.

## Lab: DOM XSS using web messages
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/77b8eafb-58e0-4972-bbdc-df76b72b1e72)
Bu labda adam direkt datayı alıp innterHTML içerisine koymuş. JSON parse falan yapmamış o yüzden daha önce oluşturulan kodu azcık modifiye edip aşağıdaki payloadı kullanmamız yeterli:  
alert(1) yerine alert(document.cookie) yazacaksın
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/8ebc077d-73a5-44a8-83aa-b99d1928e79b)
