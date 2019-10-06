# Configuring

Uygulamanın ayarları `config/application.rb` altında tutulur.

Ayrıca, 
- Environment-specific configuration files
- Initializers
- After-initializers

olarakta ayarlar yapılabilir.

Parolalar ayar dosyalarına direkt yazılmaz. Env dosyası kullanılır.

env'e yazılanlar hangi ortamda çalışıyorsanız onun için kod döner.

---

Zaman Bölgesi ayarlamak için
```ruby
config.time_zone = 'Central Time (US & Canada)'
```

?
```ruby
config.cache_classes
```

Günün başlangıç günü ayarlamak için
```ruby
config.beginning_of_week (:monday, :sunday vs.)
```

Cache'in nereye yazılacağını seçmek için
```ruby
config.cache_store (:memory_store, :file_store, :mem_cache_store, :null_store)
```

Eager load mesajlarını log'a basmak için
```ruby
config.eager_load = true
```

SSL için zorlamayı sağlamak için
```ruby
config.force_ssl = true
```

Log'ların formatını belirtmek için
```ruby
config.log_formatter
```

Log'lama seviyesi belirtmek için. Defaul olarak tüm env'lerde debugging'dir.
```ruby
config.log_level
```

---

Secret key `config/credentials.yml.enc` burada tutulur. Bunun lokasyonunu değiştirmek için
```ruby
secret_key_base
```

Pipeline'ı aktive etmek için. Default olarak true ayarlanmış gelir.
```ruby
config.assets.enabled
```

---

API/SPA bir uygulama mı yoksa Monolith bir uygulama mı ?

---

Template Engine'ı ayarlamak için
```ruby
template_engine
```

Form'da kullanılan token'i konfigüre etmek
```ruby
config.action_controller.per_form_csrf_tokens 
```

Parametreleri permit'e sokmadan çalıştırabilmek için
```ruby
config.action_controller.action_on_unpermitted_parameters
```

Genel olarak değinilen kavramlar bu şekilde. Lazım oldukça guide'dan bakmak lazım.

[https://edgeguides.rubyonrails.org/configuring.html](https://edgeguides.rubyonrails.org/configuring.html)

Action Mailer, Active Job, Database gibi şeylere bakmak lazım.

---

# Command Line

```sh
rails --help
```

---

AwesomePrint(https://github.com/awesome-print/awesome_print)

---

Sandbox sayesinde yaptığımız değişikliklerin yazılmasını engeleyebiliriz. Rails console'un özelliği.

```ruby
rails --sandbox
```

---

```ruby
rails stats
```

---

# Cache

Başına cache koyduğumuz zaman product değişmedikçe aynı kalır, bi değişiklik olduğunda alır.

```ruby
<% @products.each do |product| %>
  <% cache product do %>
    <%= render product %>
  <% end %>
<% end %>
```

Collection partial-collection kullanırken de kullanabiliriz. Aslında yukardaki ile aynı.

```ruby
<%= render partial: 'products/product', collection: @products, cached: true %>
```

### Russian Doll (Matruşka)

İç içe olduğunda da çalışır.

Örneğin bu
```html
<% cache product do %>
  <%= render product.games %>
<% end %>
```

Aşağıdaki gibi işleme girer
```html
<% cache game do %>
  <%= render game %>
<% end %>
```

Soru ?  
Touch ne ???

---

Cache'leri redis'te tutabiliriz.

```ruby
config.cache_store = :redis_cache_store, { url: ENV['REDIS_URL'] }
```

```ruby
cache_servers = %w(redis://cache-01:6379/0 redis://cache-02:6379/0)
config.cache_store = :redis_cache_store, { url: cache_servers,
 
  connect_timeout:    30,  # Defaults to 20 seconds
  read_timeout:       0.2, # Defaults to 1 second
  write_timeout:      0.2, # Defaults to 1 second
  reconnect_attempts: 1,   # Defaults to 0
 
  error_handler: -> (method:, returning:, exception:) {
    # Report errors to Sentry as warnings
    Raven.capture_exception exception, level: 'warning',
      tags: { method: method, returning: returning }
  }
}
```

---

### Caching development'da aç kapa yapılabilir.

```sh
$ rails dev:cache
# Development mode is now being cached.
$ rails dev:cache
# Development mode is no longer being cached.
```

---

https://signalvnoise.com/posts/3113-how-key-based-cache-expiration-works

http://railscasts.com/episodes/387-cache-digests

---

# Rails Action Pack Variants

https://hasantezcan.dev/blog/rails-action-pack-variant.html

---

# Rails API

Gelen isteklere cevap veren bir üründür.

Örnek olarak GitHub API, Twitter API verilebilir.

[api.github.com/users/boratanrikulu](https://api.github.com/users/boratanrikulu)

API'ler için doküman yaratmak için [swagger.io/](https://swagger.io/)


```
            REQUEST
CLIENT ------------------> SERVER

            RESPONSE
CLIENT <------------------ SERVER
```

---

API'lerde versiyonlar olur çünkü sizin api'nize bağlı uygulamalar olabilir. v1, v2, v3 diye gider...

---

Rails ile direkt olarak bir API projesi ayağa kaldırmak için
```ruby
rails new my_blog_api --api
```

Böyle yarattığımızda artık ApplicationController, ActionController::API'den türer.

---

API'nin isteklere dönebileceği noktalara Endpoint denir.

---

HTTP istekleri için POSTMAN kullanılabilir.

---

`ActionDispatch::Request#params` kullanıcıdan gelen istekleri alır.

Kullanıcıdan alınan parametreler artık controller'da params olarak kullanılabilir olur.

---

API'mızın tüm domain'lerden gelen isteklere cevap vermesi için application.rb'de

```ruby
config.middleware.insert_before 0, Rack::Cors do
  allow do
    origins '*'
    resource '*', :headers => :any, :methods => [:all]
  end
end
```

GET'in hereske açık olması mantıklı ama POST sıkıntılı. POST için araya token kontrolü girmeli. Gerçekten POST atabilir mi diye [oauth2](https://oauth.net/2/)

---

Ödev - API [Çarşambaya kadar],  

bir api projesi yazıcaz,
- Pet diye bir model olacak,  
- bu modelin name ve ownername attribute'leri olmalı,
- ownername kesinlikle olmalı(required). Eğer girilmez ise hata mesajı dönülmeli.
- crud işlemleri olacak,    
- curl ya da postman ile vs. crud işlemlerini test edilmeli.  
- [bonus] bir servis ile (ngrok gibi) dışarı açalım

not: istenilen database kullanılabilir.

---

# MVC Tekrarı

```

MVC
---
Model,
View,
Controller

             GET articles   --------
             ------------> | SERVER | >--------------------------------
            /               ---------                                  |
           /               my_wee_blog                                 |
          /                                                            |
 --------/                                                             |
| Client |                  - Article                                  |
 --------                        - Article(model)                      |
                                 - articles(controller)                |
  get '/articles', <------------ - articles(resources)                 |
       to: 'articlles#index'     - articles(view)                      |
                                    - new                              |
                                    - edit                             |
                                                                       |
                                                                       |
                                                                       |
  |--------------------------------------------------------------------|
  |
  v
  GET articles -> articles#index -> Article.all -> index.html.erb
  \
   \
    \
     \
      \
       --------
      | ROUTES |
       --------
                \                    (instance variable)
                 \                       @articles          ------                    erb yorumlanır
                  \                 ---------------------> | View | (index.html.erb) ----------------> index.html
                   \               /                        ------
                    \             /
                     ------------ 
                    | Controller |
                     ------------ 
       @articles = Article.all   \\             ----
        Article.all ile model ile \\           | DB |
        iletişim kurularak DB'den  \\           ----
        tüm article'ler çekilir     \\          /
        ve bir değişkene atılır      \\        /
                                       --------
                                      | Model  |
                                       --------

```

# Action Cable

Action Cable, WebSocket'i rails uygulamanıza sorunsuz bir şekilde entegre eder.

## WebSocket

WebSocket, kullanıcının tarayıcısı ve sunucusu arasında iki yönlü ve etkileşimli iletişim kurmayı mümkün kılan, tek bir TCP bağlantısı üzerinde çalışan gelişmiş bir iletişim protokolüdür.
WebSocket ile gerçek zamanlı bağlantılar oluşturabilirsiniz.

## Pub/Sub

Pub/Sub veya Publish-Subscribe(Yayınla/Abone ol), yayıncıların, alıcıları belirtmeden soyut bir alıcılar topluluğuna (abonelere) veri gönderdiği bir mesaj paradigmasıdır. Action Cable bu yaklaşımı sunucu ile birçok istemci arasında iletişim kurmak için kullanır.
Alıcılar bir yayıncıyı dinlemeye başlar (o yayıncıya abone olur). Yayıncılar bir veri gönderimi yaptığı zaman, tüm aboneler gerçek zamanlı olarak veriyi elde ederler.

Action Cable, hem istemci tarafı için JavaScript frameworkü, hem de sunucu tarafı için Ruby frameworkü sağlayan tam kapsamlı bir sistemdir.

İşleyiş çok basittir;

1- Sunucu tarafında bir kanal açılır.

```ruby
# app/channels/messages_channel.rb
class MessagesChannel < ApplicationCable::Channel
  def subscribed
    stream_from "messages_general"
  end
end
```

2- İstemci tarafında bu kanala abone olunur. Gelen (gelecek olan) veri ile ne yapılacağı, javascript ile halledilir.

```javascript
App.cable.subscriptions.create({
        channel: 'MessagesChannel'
    },
    {
        received: function(data) {
            // Add data to DOM.
            $('#messages-container').prepend(data.body);
        },
    });
```

3- Sunucu tarafında veri yayınlanır.

```ruby
def create
  @message = Message.new(message_params)
  if @message.save
    ActionCable.server.broadcast("messages_general", body: html(@message)
    )
  end
end

def html(message)
  ApplicationController.render(partial: 'messages/message', locals: { message: message })
end
```

Hurray :smile:

Daha özel bir senaryo için aşağıdaki gibi bir yapı kullanılabilir.

```ruby
# app/channels/messages_channel.rb
class MessagesChannel < ApplicationCable::Channel
  def subscribed
    stream_from "messages_#{params[:channel_id]}"
  end
end
```

```javascript
App.cable.subscriptions.create({
        channel: 'MessagesChannel', channel_id: $("#user_channel").text()
    },
    {
        received: function(data) {
            // Add data to DOM.
            $('#messages-container').prepend(data.body);
        },
    });
```

```ruby
def create
  @message = Message.new(message_params)
  if @message.save
    ActionCable.server.broadcast("messages_#{@message.channel.id}", html: html(message))
  end
end

def html(message)
  ApplicationController.render(partial: 'messages/message', locals: { message: message })
end
```

Hepsi bu kadar :smile:

