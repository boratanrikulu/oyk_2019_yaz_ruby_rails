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