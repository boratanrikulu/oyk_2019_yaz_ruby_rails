# Routing

devam

## Member

Yalnızca default RESTful routing'leri ile sınırlı değilsin. kendin özel de yapabilirsin.

```ruby
resources :photos do
  member do
    get 'preview'
  end
end
```

Ya da

```ruby
resources :photos do
  get 'preview', on: :member
end
```

```ruby
preview_photo GET    /photos/:id/preview(.:format)    photos#preview
```

Collection kullanılarak dönülecek kaydın çoğul yani bir collection olacağını belirtimiş oluruz.

```ruby
resources :photos do
  collection do
    get 'search'
  end
end
```

```ruby
search_photos GET    /photos/search(.:format)     photos#search
```

---

## Via

via ile çalışmasını istediğimiz method'ları topluca yazabiliriz.

```ruby
match 'photos', to: 'photos#show', via: [:get, :post]
```

---

## Subdomain

subdomain ile belli subdomain'ler geldiğin çalışmasını sağlayabiliriz.

admin.localhost:3000/photos/

```ruby
get 'photos', to: 'photos#index', constraints: { subdomain: 'admin' }
```

ya da

```ruby
namespace :admin do
  constraints subdomain: 'admin' do
    resources :photos
  end
end
```

---

## Başka path'e yönlendirmek

Başka bir path'e yönledirme de yapılabilir.

```ruby
get '/stories/:name', to: redirect('/articles/%{name}')
```

---

## Routing to Rack Applications

Başka bir aplikasyonu bir dizin isteklerine cevap verecek şekilde yapabiliriz.

Rack temelli olmalı. Yani içersinde config.ru olmalı. Http bir uygulama olmalı.

```ruby
match '/application.js', to: MyRackApp, via: :all
```

---

## Özel isimlendirmeler vermek

path'lere özel çevirileri bu şekilde yapabiliriz.

```ruby
scope(path_names: { new: 'tayfun', edit: 'erikan' }) do
  resources :categories, path: 'kategorien'
end
```

---

```ruby
rails routes --expanded
```

[http://localhost:3000/rails/info/routes](http://localhost:3000/rails/info/routes)

---

%i[index new create]

[:index, :new, :create]

-

%w[index new create]

["index", "new", "create"]

---

# Active Support

Rails'de default olarak require yapılmış gelir. 

İlla Rails'de kullanılacak diye bir şey yok. Bir ruby projesine implemente edilebilir.

İçersinde kullanabileceğimiz method'lar bulunur.

Hepsine bi bakıp neler varmış görmek lazım.  
[https://edgeguides.rubyonrails.org/active_support_core_extensions.html](https://edgeguides.rubyonrails.org/active_support_core_extensions.html)

## Try

author.books.first.title

author'un hiç book'u yok ise devamında nil hatası alırız.

```ruby
author.books.try(:first).try(:title)
```

hata oluşursa nil olarak döner.

## to_param

to_string gibi.

override ederiz.

```ruby
class User
  def to_param
    "#{id}-#{name.parameterize}"
  end
end
```

```ruby
user_path(@user) # => "/users/357-john-smith"
```

Bunun için Friendly diye bir gem var. Fakat kullanmaya pek gerek yok.

## in?

```ruby
1.in?([1,2])        # => true
"lo".in?("hello")   # => true
25.in?(30..50)      # => false
1.in?(1)            # => ArgumentError
```

## Delegate


```ruby
class User < ApplicationRecord
  has_one :profile
 
  delegate :fullname, :first_name, :last_name, to: :profile
end
```

```ruby
@user.profile_fullname
 AYNI
@user.profile.try(:full_name)


@user.profile_fullname
@user.profile_first_name
@user.profile.last_name
```

## HTML safe

html safe sayesinde içersinde bulunabilecek script'leri vs. devre dışı bırakabiliriz.

```ruby
s = "<script>...</script>".html_safe
s.html_safe? # => true
s            # => "<script>...</script>"
```

Bunun tersi raw'dır. Direkt olarak alır aynı şekilde.

```ruby
<%= raw @cms.current_template %> <%# inserts @cms.current_template as is %>
```

## String

remove -> Bir ifadeyi silmek için    
squish -> \n, boşluk vs silmek için  
truncate(n) -> n karakterden fazla ile '...' koy

## Inflections

Rails'i rails yapan kütüphanelerden.

#### pluralize

methodu ile kelimeleri çoğul yapabiliriz.

```ruby
"table".pluralize     # => "tables"
"ruby".pluralize      # => "rubies"
"equipment".pluralize # => "equipment"
```

```ruby
"dude".pluralize(0) # => "dudes"
"dude".pluralize(1) # => "dude"
"dude".pluralize(2) # => "dudes"
```

#### singularize

```ruby
"tables".singularize    # => "table"
"rubies".singularize    # => "ruby"
"equipment".singularize # => "equipment"
```

#### classify

kelimenin class ismi karşılığını gösterir.

```ruby
"people".classify        # => "Person"
"invoices".classify      # => "Invoice"
"invoice_lines".classify # => "InvoiceLine"
```

#### foreign_key

ile foreign key karşılığını görebiliriz.

```ruby
"User".foreign_key           # => "user_id"
"InvoiceLine".foreign_key    # => "invoice_line_id"
"Admin::Session".foreign_key # => "session_id"
```

## Time

```ruby
# equivalent to Time.current.advance(months: 1)
1.month.from_now
 
# equivalent to Time.current.advance(weeks: 2)
2.weeks.from_now
 
# equivalent to Time.current.advance(months: 4, weeks: 5)
(4.months + 5.weeks).from_now
```

## Phone

String ifadesini telefon olarak formatla

```ruby
5551234.to_s(:phone)
# => 555-1234
1235551234.to_s(:phone)
# => 123-555-1234
1235551234.to_s(:phone, area_code: true)
# => (123) 555-1234
1235551234.to_s(:phone, delimiter: " ")
# => 123 555 1234
1235551234.to_s(:phone, area_code: true, extension: 555)
# => (123) 555-1234 x 555
1235551234.to_s(:phone, country_code: 1)
# => +1-123-555-1234
```

## Currency

```ruby
1234567890.50.to_s(:currency) # => $1,234,567,890.50
1234567890.506.to_s(:currency) # => $1,234,567,890.51
1234567890.506.to_s(:currency, precision: 3) # => $1,234,567,890.506
```

## Human readable

```ruby
100000000000000000.to_s(:human)
```

```ruby
1234567890123.to_s(:human_size)
```

## Date

```ruby
d = Date.new(2010, 5, 8)     # => Sat, 08 May 2010
d.beginning_of_week          # => Mon, 03 May 2010
d.beginning_of_week(:sunday) # => Sun, 02 May 2010
d.end_of_week                # => Sun, 09 May 2010
d.end_of_week(:sunday)       # => Sat, 08 May 2010
```

# Rails Internationalization (I18n) API

```
Internationalization
|------------------|

I   18 tane harf   n
```

Arayüz Çevirileri,  
URL Çevirileri

işlerini yapar. Kayıt çevirileri başka bir şeydir. Onun i18n ile ilgisi yok.

config/locales/en.yml

key-value şeklinde bir dosya. çevirilerimizi bu dosyaya yazıyoruz.

```ruby
en:
  hello: 'hello'
```

Kullanacağımız zaman da `t('hello')` şeklinde kullanırız.

bunun türkçe çevirisi yapmak için de config/locales/tr.yml dosyasında

```ruby
tr:
  hello: 'merhaba'
```

Artık sistemin dili ing ise hello, türkçe ise merhaba yazacaktır.

Dili set'lemek için application.rb'ye direkt olarak `config.i18.default_locale = :tr` yazabilirdik. Ama bundan sonra server aç kapa yapmak vs. gerekiyor, pek kullanılabilir bir yol değil.

Bunun için application controller'a ir method ekleriz.

kullanıcı artık localhost:3000?locale=tr path'ine giderse tr görecektir. Ama her defasında basmasını bekleyemeyiz, bundan dolayı cookie'ye yazmamız gerekecek kullanıcının tercihini.

---

URL'den dil tercihini almak için.

Application controller default url option set'leriz.

```ruby
def default_url_options
  { locale: I18n.locale }
end
```

Tüm controller'i bir scope'a almak için application controller'ı scope alıyoruz.

```ruby
Rails.application.routes.draw do
  scope "/:locale" do
    # For details on the DSL available within this file, see https://guides.rubyonrails.org/routing.html

    # Post actions
    resources :posts
    resources :users
    resources :birkelime

  	resources :publishers, only: [:index, :new, :create] do
  	  resources :magazines, only: [:index, :show] do
  	    resources :photos, only: [:index]
  	  end
  	end

    resources :photos do
      collection do
        get 'search'
      end
    end

    post 'users/create_session', to: 'users#create_session'
    post 'users/delete_session', to: 'users#delete_session'
  end
end

```

---

Kullanıcıdan alınan tercihi link üzerinden cookie'e setlettirmeye bakman lazım.

---

Çeviriler'de değişkenler kullanılabilir.

```ruby
tr:
	hello: "Hello %{name}"
```

```ruby
t('hello', name: 'bora')
```

---

### Date Time Çevirileri

Bunun için direkt olarak rails-i18n gem'ini kurabiliriz. İçersinde bir çok dil var.

```ruby
<%= l Time.now, format: :long %>
```

---

### Model Katmanındaki şeylerin çevrilmesi

Model isimleri, hata mesajları vs.. filan hepsini çevirebiliriz.

```ruby
en:
  activerecord:
    models:
      user:
        one: Customer
        other: Customers
```

# Action Mailer

Uygulama üzerinden mail atmak istediğimizde kullanıyoruz. Örneği bir kullanıcı kayıt olduğunda onaylama epostası gideceğinde kolayca eposta yollayabilmemizi sağlar.

Eposta yollayıcının yaratılması
```ruby
rails generate mailer UserMailer
```

```ruby
# app/mailers/application_mailer.rb
class ApplicationMailer < ActionMailer::Base
  default from: "from@example.com"
  layout 'mailer'
end
 
# app/mailers/user_mailer.rb
class UserMailer < ApplicationMailer
end
```

Eposta'nın yollanma aşaması
```ruby
class UserMailer < ApplicationMailer
  default from: 'notifications@example.com'
 
  def welcome_email
    @user = params[:user]
    @url  = 'http://example.com/login'
    mail(to: @user.email, subject: 'Welcome to My Awesome Site')
  end
end
```

View dosyasının oluşturulması
```html
<!DOCTYPE html>
<html>
  <head>
    <meta content='text/html; charset=UTF-8' http-equiv='Content-Type' />
  </head>
  <body>
    <h1>Welcome to example.com, <%= @user.name %></h1>
    <p>
      You have successfully signed up to example.com,
      your username is: <%= @user.login %>.<br>
    </p>
    <p>
      To login to the site, just follow this link: <%= @url %>.
    </p>
    <p>Thanks for joining and have a great day!</p>
  </body>
</html>
```

Rails'in eposta yollayabilmesi için SMTP ayarlarının yapılı olmalı. Rails tek başına mail atabilir değil. SMTP sunucusuna epostayı teslim edebiliyor yalnızca.

Ayarları set'lemek için.
```ruby
config.action_mailer.delivery_method = :smtp
config.action_mailer.smtp_settings = {
  address:              'smtp.gmail.com',
  port:                 587,
  domain:               'example.com',
  user_name:            '<username>',
  password:             '<password>',
  authentication:       'plain',
  enable_starttls_auto: true }
```

Test amaçlı Mailtrap kullanılabilir.

# Asset Pipeline

Rails 6 ile birlikte Asset Pipile'nin yerini Webpacker almaya başladı.

Webpacker'de assets dizininde bulunmaz. Direkt app'in altında bulunur.

Asses Pipeline'da ise assests/javascript, asset/stylesheet, asset/images

---

## image_tag

app/assets/images
```ruby
<%= image_tag "rails.png" %>
```

image_tag kullanınca gösterim sırasında fotoğraf adına bir hash ekler. Bunun amacı cache'leye bilmesi içindir. Fotoğraf değişince hash'de değişir fotoğrafın değiştiğini anlamış olur.

css içinde ruby kodu çalıştırmak için .css.erb demek yeterli olur. Örneğin css'de arkaplan fotoğrafı ekleyeceğimizde bunu kullanırız.

```css
.class { background-image: url(<%= asset_path 'image.png' %>) }
```

---

Normal şartlarda veriler asset altındayken proje ayağa kalacapında sıkılaştırılarak public'e atılır.

```ruby
RAILS_ENV=production rails assets:precompile
```

---

webpack ile react, stimulus gibi şeylerin direkt olarak kurulmasını sağlayabiliriz.

```ruby
# Rails 5.1+
rails new myapp --webpack=react
```

Hali hazırdaki bir projeye kurmak için
```ruby
rails webpacker:install:stimulus
```

---

SPA (single page app) framework'lerinden rails'e en yakını ember.js (https://emberjs.com/)

---

# Stimulus.js

Projemize kuralım.
```ruby
rails webpacker:install:stimulus
```

Bunu yaptığımızda javascripts/ klasörüne controllers diye bir klasör geldi.

içersinde hello_controller.js
```js
import { Controller } from "stimulus"

export default class extends Controller {
  static targets = [ "output" ]

  connect() {
    this.outputTarget.textContent = 'Hello, Stimulus!'
  }
}
```

Users/new'e yaptığımızda hello controller'ı çalışacak ve output controller set'lendiği gibi gözükecek.
```html
<!--HTML from anywhere-->
<h2>Stimulus</h2>
<div data-controller="hello">
	<span data-target="hello.output">
  </span>
</div>
```

Alert diye bir method ekleyip bir button koyarsak.

```js
import { Controller } from "stimulus"

export default class extends Controller {
  static targets = [ "output" ]

  connect() {
    this.outputTarget.textContent = 'Hello, Stimulus!'
  }

  alert() {
  	alert('Alerrrrtttttttt!')
  }
}
```

```html
<div data-controller="hello">
  <span data-target="hello.output">
  </span>
  
  <button data-action="click->hello#alert">
    Alert
  </button>
</div>
```

# Rails'de JavaScript

Rails javascript isteklerini halletmek için Unobtrusive JavaScript denen bir teknik kullanır.

```html
<input type="checkbox"
       data-remote="true"
       data-url="<%= welcome_path %>"
       data-params="id=10&another=2"
       data-method=put
```

...   
...   
...  
...  

Bu kısmın notunu alamadım. [https://edgeguides.rubyonrails.org/working_with_javascript_in_rails.html](https://edgeguides.rubyonrails.org/working_with_javascript_in_rails.html)

Basitçe ajax atma örneği yapıldı.

---

Stimulus kullanarak kullanıcıdan dil set'lettirme örneğini yaptık.

---

# Turbolinks

Temelini GitHub'ın eski ceo'su atıyor. Basecamp'ciler alıp devam ettiriyor.

Sayfanın yüklenme hızını artırır. Her sayfaya girildiğinde html/css yenilenmesinin önüne geçmiş olur.

IOS ve Android adaptörleri var. native gibi uygulama yazılabilir.

Android versiyonu şuan geliştirilmiyor.!

---

Mobil tarafına değil web tarafına bakıcaz.

---

GitHub'da da kullanılıyor.

---

Tüm link'ler sanki ajaxmış gibi çalışır.

**Sadece değişen kısımlar güncellenir.**

---

History'da kalınan yerleri filan da tutar.

Turbolinks tarayıcının cache mekanizmasını yönetir.

Sürekli ajax isteği attığı için kendi içinde bir yükleme barı gösterir.

---

Rails'de form'ları ajax haline getirilirse, hedeftedi action'da js.erb kullanılmasına gerek kalmaz. Direkt redirect_to yapılır geri kalanı turbolinks otomatik olarak halleder.

---
