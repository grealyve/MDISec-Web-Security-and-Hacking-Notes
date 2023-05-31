<h1 align="center">CS253 vs MDISEC</h1>

- Session Hijack :
  1) Bir kullanıcının cookiesini çalarak hijack edebilirsin
  2) Cookiesini tahmin ederek hijack edebilirsin
  3) XSS exploit edip HTTP-ONLY set edilmemişse oturum anahtarını çalarak da session hijack yapabilirsin.

- HTTP Response’da neden detaylı error infosu vermek yanlıştır?
  - Runtime env variable expose olabilir
  - local path bilgisi ifşa olabilir
  - kullanılan library versions info leak olabilir.
  - partial source code leak olabilir
## Soru 6:
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/93d5d89b-89e8-4b6e-bc22-7785808f7cb7)
- Bu da cevabı:
- ![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/44eb6230-c80c-4163-b454-8d6c56655069)
## Soru 8:
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/daae59f6-47ea-41bb-93fc-acd149b65ec3)
- unsafe-inline yazmayınca aktif oluyormuş zati
- Cevabı:
- ![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/e65d8bb1-1d62-4c3e-9e38-e8f5a2bb9641)
## Soru 9:
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/ba3a32c2-b807-4718-976b-aefb52c14916)
- Cevabı:
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/eaf2d669-683d-428e-92cf-eaf61f373d02)
## Soru 10:
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/e221e827-638a-4b96-86b2-b6925512cab9)
- Captcha rate limiting tetiklenince çalışır zaten.
- Cevabı:
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/c002e8e6-cebd-4e9a-a454-e50a4b99fb07)
## Soru 11:
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/0ecd2213-c317-45f0-b48c-2cb5c94d1508)

# Free Response
## Soru 1 - Same Origin Policy
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/cd0146bd-cb12-46fb-8577-908b72fc0091)
## Soru 3 - CORS Preflight
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/a5dbd51b-ca87-4f57-ae42-98f15d34a4b0)
- Cevap: JS ile bir API’den data alman ile alakalıdır. API’ler genellikle REST-API standartlarına göre ayarlandığı için, bana bu resource ile ilgili ayarları ver dediğin bir request gönderirisin. Ben gerçek bir request göndericem, bundan öncesinde bu kaynak ile konuşabiliyor muyum ona bakarsın.
## Soru 4 - Cookies:
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/1ca51631-5470-4072-a116-c9dba2a6734d)
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/30d459f8-26d3-48da-8e1b-83d2d668df0d)
- Cevap: CORS aynı olduğu için attacker hedef siteyi iframe içerisinde açıp, contentine erişim sağlayabilmekte. Çünkü CORS için scope yani path’in bir önemi yok.
## Soru 5 - More Cookies:
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/244a4ae5-c969-4165-80ca-ad3390b3d57b)
- Banka, tespiti referer ile yapıyor. Browser otomatikman referer headerı ekler img’deki isteğe ve  banka uygulaması da bu bilgiyi kontrol eder kendi kaynağından mı gelmiş bu istek diye acaba?
- Cevap:
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/d7333d38-05c4-4bb6-a3de-717373edc5d8)
## Soru 6 - XSS:
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/fdc0ba96-8c7e-4aee-9f14-2f29b965b2be)
- query string’de GET parametresi ile giden source’u alıp header’a koymuş. HTML context XSS var.
- Cevap:
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/f911e224-2d82-4eda-8715-f71469577538)
- username alanında XSS payloadını yazacaksın. Aşşağıda script tagine koyuyor senin username’ini, tek tırnak ile (’) escape edip koyabilirsin encode edilmediyse. Eğer tek tırnak encode edildiyse script tagini kapatıp payloadı yazarsın.
- Alert’in içinde de XSS case var.
```javascript
</script><svg onload=alert(1)>
```
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/7f2586e0-4c60-45ea-a38c-ca89c240262b)
## Soru 7 - More XSS
- Cevap:
- ![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/9b69fb04-b719-45f1-a665-78b4eb331266)
- back-slash, back-slash’i escape ettiği için tek tırnak escape edilemiyor
- ![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/a6007cfa-1197-4605-86de-97e6e3b63cce)
## Soru 8 - CSP:
- Soru: Bu CSP kuralına göre aşağıdakilerden hangisi yüklenir/yüklenmez?
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/f54d1f8c-06cf-456f-bd80-041c12b08541)
- style-src sadece self yazdığı için 2.si F olur yüklenmez.
- script-src self olduğu için alert çalışır.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/9a4c040b-1492-4506-b5b8-ea9d7ad81fa8)
- Cevap:
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/5feb55ec-e378-483b-a96e-9be991c5d3b6)
- Daha önceki kafa karışıklığına sebep olan unsafe-inline muhabbeti yüzünden yanlış.
## Soru 9 - HSTS Preload:
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/21ee0398-ce70-44c4-b3a2-d83ce513c55d)
- Preload list’e koyduğun zaman tüm browserlarda HTTP request gitse bile 307 Internal Redirect ile bu request HTTPS’e yükseltilir ve trusted auth. tarafından doğrulama yapılır falan..
- Cevap:
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/a155f6c5-9aae-4b47-90a3-34f1c0754a22)
## Soru 10 - Command Injection:
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/72aad012-e3b2-4bca-b924-4ba9f2d3bb06)
- Cevap:
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/28f1f5ca-6abc-4d34-b3c9-e99b2c850c7a)
## Soru 11 - Fingerprinting:
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/c283a861-cb23-441e-a612-43ecef83b5f1)
- Cevap:
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/152da462-7dce-4cbd-a7f7-5f731d4e9632)
## Soru 12 - Logic Bug:
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/729804d1-45a0-48d8-ac0c-5e1d3ecac62c)
1) CSRF,
2) no return/exit : kullanıcı adı parola vermesen bile çalışır. Çalıştıktan sonra fonksiyon kapanmaz, sadece ekrana bir şey yazar.
- kritik metodlar GET ile gelmemeli diyor.
![image](https://github.com/grealyve/MDISec-Web-Security-and-Hacking-Notes/assets/41903311/206c0a1e-9c89-4f0c-a7b2-7e345933e9fa)
