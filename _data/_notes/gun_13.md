# Web Socket Tekrarı

- Pub/Sub -> Yayıncı/Abone

```

 ----------               --------
| Tarayıcı | <---------> | Sunucu |
 ----------               --------
  HTML5
  Web Socket

|____________|          |_____________|

  Abone                    Yayıncı
                                     -----
  - Connection             - Connection   |  Kanals
  - Subscribrtion          - Broadcast    |
                                     -----
```

Örnek olarak chat grupları verilebilir. Char grubu broadcast yapar. Kullanıcılar abonelerdir. Yeni mesaj var ise broadcast'te yayınlanır.

Eskiden, web socket yok iken kullanıcıların belli aralıklarla yeni mesaj var mı diye sorması gerekirdi.

Web socket **`ws://`** portunda çalışır.

---

Web Socket'i Rails tarafında yapmak için **Action Cable** kullanılır.

---

https://aws.amazon.com/tr/sns/  
https://pusher.com  
https://onesignal.com/  
https://firebase.google.com/products/cloud-messaging/

---

# Active Storage

Attachment, yani etkiketlemek. Genellikle bir dosya olur. Resource'a attach edilir.

User, attach: profile_picture  
User, attachments: licenses

Active Storage'dan önce en çok Papper Clip kullanılıyormuş.  

---

Storage'i local'de de tutabilir, cloud'da da.

---

Cloud Based Storage Server'lar. (Ücretliler. Özgür değiller.)  

Amazon S3  
Azure  
Google Storage

---

https://az.minio.io/

---

# Active Job - Arkaplan Görevleri,

https://edgeguides.rubyonrails.org/active_job_basics.html  
https://boratanrikulu.dev/active-job-rails-zamanlanmis-gorevler/   

Cron görevleri için  
https://github.com/javan/whenever  
https://github.com/Moove-it/sidekiq-scheduler  

---

# Rails Template

**`.env`**

https://github.com/bkeepers/dotenv

---

# Rails Engine

Kebab project.

**Cybele**

---

# Ortak Çalışma Alanı Yönetim Sistemi

MVP olarak sistemi

Kişisel repo'da private,  
README.md,  
VERSION.txt,  

Hiç bir gem yok,  
Bootstrap yok,  
Bir yere deploy yok,  
Grup'lar arası iletişim yok

---

Mekana gelmeden önce başvuru yapacak. (ne amaçla kullanacaklar, haftanın hangi günleri kullanacak ?, adı soyadı ve eposta ve kısözgeçmiş)

Yönetici bunu değerlendirip kabul edebilcek. Eğer kabul edilirse kullanabilecek.

- Kullanıcılar (abone)
  - Başvuru
    - İsim
    - eposta
    - parola
    - bio
    - ne amaçla kullanacak
  - Başvuru durumunu görebilir
  - Oturum açabiliyor
  - Kapı için PIN üret tuşu ****
- Paydaşlar (ünvanı, eposta adresi, parola)
  - Başvurular önüne düşecek
  - Onayla / Onaylama / Kararsız
  - Toplam onaylnan oy sayısı paydaş sayısının 50% ise üyelik onaylansın.
  - Sistemdeki üyeleri görebilir
  - Verdiği oyu değiştiremez
  - Giriş bilgilerini değiştirebilir
- Yönetici
  - Başvuru sürecine dokunmak istemiyor
- Anasayfa
	- İçerde şuan 15 kişi var

---

# Mehmet İnce - Web'de Güvenlik

HTTP

TCP

3'lü el sıkışma

Syn/ack paketleri

---

Her request-response öncekinden bağımsızdır.

---

```
                             --
(direktif veren kısım)  Header |
                        Body   | ------------- Her Request ve Response iki kısımdan oluşur.
                             --
```

---

Tarayıcılar genellikle cache'leri sqlite'da tutar.

---

Cookie'lar request'lere tarayıcılar tarafından otomatik olarak eklenir.

---

Session uygulama çalışırkan client'in, server tarafından tanınabilmesini sağlar.

```
           Cookie               Session
          --------             ---------
           Client               Server
```

Yani cookie'de tutulan token ile kullanıcı siteye giriş yapmış olur. Her request'de bu token isteğe ekleneceği için sunucu kişinin kim olduğunu anlar.

Yani kişinin logout olması için session'ın yok edilmesi yeterlidir. Sunucu kullanıcıyı tanımaz hala gelir ve logout olunmuş olur.

---

Cookie'ler set'lenirken flag'ler kullanılır.

Örneğin path diye bir flag vardır. Bu flag sayesinde cookie'nin hangi path'lerde çalışacağını belirtebiliriz.

```
set cookie isim=...............................
..............; path="/", HttpOnly
                    -----
                      |
                      |_> Yani bu cookie tüm sitede çalışacaktır(geçerlidir).
```

Rails secretket ile tüm cookie'leri imzalar. Bu sayede cookie'lerin değişip değişmediğini anlamış olur.

---

# Cross-Site-Request-Forgery (CSRF)

Kullanıcı isteği gerçekten isteyerek mi attı ? Örneğin sahip olduğu tüm verileri silecek bir istek ama gerçekten bu isteği atan kişi o mu yoksa girdiği bir sitede onun adına bir istek mi atıldı ?

Bunun için gerçek sitede istek atılacak yerlere bir hidden field eklenir ve burada cookie'da tutulan değer ile aynı bir token değeri yazılır. Yani her istek gittiğinde bu değer de formda bir hidden field içersinde karşıya aktarılır. Sunucu tarafında da bu şekilde token içeriği olmayan isteklerde çalışmayacak şekilde ayarlar isek, başak sitelerden istek geldiğinde bu şekilde token yollanmayacağı için bunu sorunu çözmüş oluruz.

Bu rails'da otomatik olarak yapılır. Yani rails'de csrf koruması vardır.

Tarayıcılara yeni bir cookie flag'ı gelmekte.. Bu flag sayesinde cookie'nin yalnızca ilgili sitelerden istek atılırken isteğe atılması sağlanacak ve bu csrf açığı ilerde mevzu bahis olmayacaktır. Bu flag'in adı `SameSite`

---

# JWT (json web token)

```
 ---------             -------
| Session | --------> | Redis |
 ---------             -------
                     Daha hızlı sorgu atarız
```

Kişinin kimliği bir json objesinde tutulur. İçersine key'de konur. Eğer içindeki bişey değişirse keyden farklı olur değiştiği anlaşılır. Kişi bu obje ile gittiği heryerde kendini tanıtabilir.
