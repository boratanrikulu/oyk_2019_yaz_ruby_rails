## Query Interface

devam

## Unscope

Daha önceden yazılı olan scope'den bir query'i çıkarmak için kullanırız.

```ruby
Article.where('id > 10').limit(20).order('id asc').unscope(:order)
```

## Reorder

Varsayılan olarak sıralama şeyini değiştirmek için kullanılır.

```ruby
class Article < ApplicationRecord
  has_many :comments, -> { order('posted_at DESC') }
end
 
Article.find(10).comments.reorder('name')
```

---

## Joins

İncludes'da ayrı ayrı tutar. Tek bir seferde tüm veriyi çekmek içindir. Bunda ise direkt tek bir tabloda çekilir veri.

Çekmek istediğimiz veriyi tek bir collection gibi alabilmemizi sağlar.

Article ile category ve comments arasında ilişki var ise.
```ruby
Article.joins(:category, :comments)
```

Article ile comments, comments ile de guest arasında ilişki var ise.
```ruby
Article.joins(comments: :guest)
```

Article'e bağlı comments ve tags'i çağır, ve comment'e bağlı guest'i
```ruby
Category.joins(articles: [{ comments: :guest }, :tags])
```

---

## Eager Loading Associations

n+1 

n+1 hatalarını tespit etmek için Bullet.

```ruby
clients = Client.includes(:address).limit(10)
 
clients.each do |client|
  puts client.address.postcode
end
```

n+1 hatalarının önüne geçmek için **includes**

---

## Scopes

Kodun okunmasını artırır.  
Daha sonra kullanılmak üzere yazdığımız sorgulara bir isim vermektir.  

```ruby
Article.where(published_true)
```

Yerine
```ruby
scope published, -> { where(published_true) }
```

Bundan sonra dememiz yeterlidir.
```ruby
Article.published
```

Scope içinde scope'da kullanabilir.

```ruby
class Article < ApplicationRecord
  scope :published,               -> { where(published: true) }
  scope :published_and_commented, -> { published.where("comments_count > 0") }
end
```

Scope'a parametre göndermek için.
```ruby
class Article < ApplicationRecord
  scope :created_before, ->(time) { where("created_at < ?", time) }
end
```

Scope'lara koşul eklemek
```ruby
class Article < ApplicationRecord
  scope :created_before, ->(time) { where("created_at < ?", time) if time.present? }
end
```

Fat Model, Skinny Controller.

Defualt scope ile varsayılan bir scope oluşturabiliriz.

```ruby
class Client < ApplicationRecord
  default_scope { where(active: true) }
end
```

```ruby
Client.new          # => #<Client id: nil, active: true>
Client.unscoped.new # => #<Client id: nil, active: nil>
```

## Dynamic Finders

Sistemi yoran birşey. Pek fazla kullanmamak gerekiyor.

```ruby
Client.find_by_first_name_and_last_name("Bora", "Tanrıkulu")
```

## Enums

```ruby
class Person < ApplicationRecord
  enum gender: [:male, :female]
end
```

```ruby
Person.male

Person.first.female?
```

## Find or create by

Bulursa getirir. Bulmaz ise önce oluşturup sonra getirir.
```ruby
Client.find_or_create_by(first_name: 'Andy')
```

## Find or initialize by

New demiş getirir, kaydetmez
```ruby
Client.find_or_initialize_by(first_name: 'Nick')
```

## Find by sql

SQL sorgusunu elimizle yazabiliriz.

```ruby
Client.find_by_sql("SELECT * FROM clients
  INNER JOIN orders ON clients.id = orders.client_id
  ORDER BY clients.created_at desc"
```

## Pluck

Bir collection'dan seçtiğimiz attribute'lere göre array almak için.

```ruby
Client.pluck(:id)
```
```ruby
Client.pluck(:id, :name)
```

Array'lerdeki map özelliği ile aynı. Pluck yapmasaydık.

```ruby
Client.select(:id).map { |c| c.id }
```

Bunun kısa hali de 
```ruby
Client.select(:id).map(&:id)
```

```ruby
Client.select(:id, :name).map { |c| [c.id, c.name] }
```

Getter method'u ezerek map'de kullanabilir. Pluck'da olmazdı bu.

```ruby
class Client < ApplicationRecord
  def name
    "I am #{super}"
  end
end
```

```ruby
Client.select(:name).map &:name
# => ["I am David", "I am Jeremy", "I am Jose"]
 
Client.pluck(:name)
# => ["David", "Jeremy", "Jose"]
```

ids ile pluck(:id) yapılır
```ruby
Person.ids
```

## Exist ile var mı yok mu diye bakılabilir.

```ruby
Client.where(first_name: 'Ryan').exists?
```

```ruby
Client.exists?(id: [1,2,3])
# or
Client.exists?(name: ['John', 'Sergei'])
```

## Any - Many

Hiç var mı ?  
Çokça var mı ?

```ruby
Article.any?
Article.many?
```

## Explain

Explain ile sorguya bir tablo dönebilir.
```ruby
User.where(id: 1).joins(:articles).explain
```

# Layouts and Rendering(View)

User intreface kısmını oluşturduğumuz bölüm.

Template  
HTML  
Önyüz  
UI

```
 ----                                 İSTEK
| DB |                               /
 ----                               /
  ||                        --------
  ||                       | Router |
 -------                    --------
| Model |                /
 -------                /
   \\                  /
    \\                /
     \\   ------------               ------
      \\ | Controller | <---------> | View |
          ------------               ------
```

## Render

En önemli araç.

Render yapıldığında bir **response** oluşturulur.

Convention over Configuration.

Controller'ın adı ne ise view'da da öyle olur. Default olarak orası render edilir.

```
index.html.erb
 |     |     |
isim uzantı derleyici
```

index.css.erb, index.js.erb vs de olabilirdi.

çıktının ekranda gözükmesi için `<%= %>`, sadece çalışması için `<% %>`

---

Render controller'da action'ların view'lerini render'lar. Default olarak aynı isimdeki dosyayı render'lar.

```ruby
def update
  @book = Book.find(params[:id])
  if @book.update(book_params)
    redirect_to(@book)
  else
    render "edit"
  end
end
```

Her action bir view üretecek diye bir şey yok. Çıktı üreten ve çıktı üretmeyen(bir işlem yapıp yönlendiren) action'lar vardır.

---

Rails'de view dosyalarına template denir

---

```ruby

render :edit
render action: :edit
render "edit"
render "edit.html.erb"
render action: "edit"
render action: "edit.html.erb"
render "books/edit"
render "books/edit.html.erb"
render template: "books/edit"
render template: "books/edit.html.erb"
```

---

## İnline

Direkt olarak inline yazabilmemizi sağlar.

```ruby
render inline: "<% products.each do |p| %><p><%= p.name %></p><% end %>"
```

## Plain

Plain, raw data olarak dönmek için kullanılır.

```ruby
render plain: "OK"
```

## Json

```ruby
render json: @product
```

## XML

```ruby
render xml: @product
```

## JS

JS dönebiliriz. Bunu döndüğümüzde browser anlayıp otomatik olarak çalıştırır.

```ruby
render js: "alert('Hello Rails');"
```

```
This will send the supplied string to the browser with a MIME type of text/javascript.
```

**MIME type**

hangi tipte veri olduğunu belirtir.

## Render için seçenekler

- content_type
- layout
- location
- status
- formats

**content_type**

```ruby
render file: filename, content_type: "application/rss"
```

**layout**

```ruby
render layout: "special_layout"
```

```ruby
render layout: false
```

**location**

```ruby
render xml: photo, location: photo_url(photo)
```

**status**

sunucunun dönen veri hakkında verdiği açıklayıcı bilgi.

```ruby
render status: 500
render status: :forbidden
```

**variant**

```ruby
ruby variants: [:mobile, :desktop]
```
+
```
views/home/index.html+mobile.erb
views/home/index.html+desktop.erb
views/home/index.html.erb
```

Eğer farklılıklar ufak ölçekte ise direkt olarak CSS ile yapabiliriz. Ama tasarım büyük ölçüde değişiyor ise bu durumda variant kullanmamız gerekiyor. Yani Mobile'de başka bir sayfa, desktop'da farklı bir sayfa gösteriyorsak bunu variant ile yapabiliriz.

---

## Layout

Sayfanın bütünün gösterildiği kısımdır aslında.

Her sayfada aynı kalan, değişmeyen kısımlar genellikle layout'a taşınır.

Default olarak ApplicationLayout bulunur.

Controller'da layout'ı ayarlamak için
```ruby
class ProductsController < ApplicationController
  layout "inventory"
end
```

Duruma göre farklı layout kullanmak için
```ruby
class ProductsController < ApplicationController
  layout :products_layout
 
  def show
    @product = Product.find(params[:id])
  end
 
  private
    def products_layout
      @current_user.special? ? "special" : "products"
    end
 
end
```

Action'a göre farklı layout için
```ruby
class ProductsController < ApplicationController
  layout "product", only: [:index, :show]
end
```

Layout kullanmak için
```ruby
class OldArticlesController < SpecialArticlesController
  layout false
end
```

## Redirect_to

Render'dan farklı olarak bunda farklı bir controller-action'a istek aktarılır-atılır.

## Flash

Hata mesajları bastırmak için flash kullanılır.

Flash mesajları kullanıldığı an mesajı bellekten siler.

```ruby
def index
  @books = Book.all
end
 
def show
  @book = Book.find_by(id: params[:id])
  if @book.nil?
    @books = Book.all
    flash.now[:alert] = "Your book was not found"
    render "index"
  end
end
```

## Head Request

API yazarken head request yazmamız gerekebilir.

```ruby
head :bad_request
```

```ruby
head :created, location: photo_path(@photo)
```

---
