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


