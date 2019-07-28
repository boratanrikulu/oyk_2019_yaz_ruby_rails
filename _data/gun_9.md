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


