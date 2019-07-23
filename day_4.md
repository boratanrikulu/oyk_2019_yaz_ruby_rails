# ÖYK 2019 Ruby on Rails - Day 4

* *[cloudflare](https://www.cloudflare.com/)* - Örnek bir Domain Name Server

Request pdf için gelir ise, accept_language pdf olarak, html set'lenir ise text/html olarak geri dönüş yapılır.

## Rack

https://rack.github.io/

Ruby'nin http isteklerini anlayabilmesi için araya *rack* katmanı girer.

`Rack provides a minimal interface between webservers that support Ruby and Ruby frameworks.`

Herhangi bir dizinde *config.ru* var ise o bir **webserver**'dır. Gelen isteklere cevap verebilir.

Proje ayaklandığında **puma** adında bir web sunucusu çalışır.

puma (application web server) (http://puma.io/)
thin ruby webserver
ruby unicorn

```ruby
# config.ru
require './myapp.rb'
run myapp
```

```ruby
# myapp.rb
require 'rack'
require "rack/handler/puma"
 
myapp = Proc.new do |env|
    ['200', {'Content-Type' => 'text/html'}, ['A barebones rack app.']]
end
 
Rack::Handler::Puma.run myapp
```

```ruby
rackup
```

Development'dayken **app server**, Production'da ise webserver kullanılır direkt olarak app server kullanılır.

**nginx** statik veriler, ssl, saldırılardan sorumlu olarak ayarlanır genelde, puma ve alternatif app serverlar ise ruby kodları çalıştırmak için vs.

Nginx aslında bir proxy görevi görmüş olur.

CI

Session, cookie, sunucunun client'ı tanımasını sağlar.

## MVC

```
 _______________
|     || NGINX  | <---------------- Request (client)
|     ||        |
|     || PUMA   |
|     ||        |
|     || APP    | -----> - public/
|     ||        |            - public/assets
|     ||        |        - app/
|     ||        |            - app/controllers
|     ||        |        - config.ru
 ---------------         - routes.rb
    |
    |
    |
    |
    |
    --------------
   |              |
   |  PostgreSQL  |
   |              |
    --------------


```


```
  -------                  ------
 | Model |                | View |
  -------                  ------
    \                         /
     \                       /
      \                     /
       \                   /
         ------------------
        | Controller       |
        | Routing          |
        | Front Controller |
         ------------------
```

```
- Design Pattern
	- FrontController
		- Routing   <----------------| Dispatch
		- Controller Classes --------|
		- URI
```

FrontController adında bir design pattern var. MVC'lerin hepsi başlangıçta FrontController uygulayarak başlar. Bu routing ve controller class'larının ilişkili olarak çalışmasını sağlar.

**HTTP Request Methods** [REST ile doğrudan ilgisi var!]
```
GET
POST
PUT/PACTH
DELETE
```

FrontController root path'den sonraki alt path'leri algılar ve nereye yani hangi controller'a istek yapılması gerektiğini anlar, yani dispatch yapar!

Yapılan http request tipine göre de controller'ın hangi method'u yani hangi action'ı çalışacağı anlaşılır.

Controller'a gidilten sonra eğer DB üzerinde bir işlem yapılacak ise Model katmanı araya girer ve DB üzerinde ilgili sorgular yapılır. (SQL)

SQL sorguları iki gruba ayrılır. SQL aslında bir DSL'dir yani Domain Specific Language.

**DFL** - Data Defination Language
**DML** - Data Manupilation Lanugage

Rails'de SQL sorguları model üzerinde otomatik olarak oluşturulur. Bunu ORM sağlar yani Object Relation Mapping.

---

Model katmanında yapılacak sorgu işlemleri yapıldıktan sonra controller view katmanınına gider.

### View

* assets
* layout
* template
* render

controllers/users_controller.rb  
views/users/index.html.erb  
models/user.rb

## Active Record

Convention over Configuration.

Kurallara uyulmayabilir ama bu durumda hata oluşabilir.

Keskin bıçaklar. Standartları kendine göre değiştirebilirsin, ama bu durumda bıçak üstünde olduğunu unutma!

```
    DATABASE
 --------------
|  PostgreSQL  |
|  SQLite      |
|  MySQL       |
|  MongoDB     |
 --------------
```

```

DB (host, username, password)                       ORM (Active Record)
  -> Scheama
     -> Table   <----------------------------->  Class
       -> Column  <------------------------>  Attribute
         -> Type
```

```
       -----
      | ORM |
       -----
         |
         | Connection
         |
         |
     -----------------
    | PostgreSQL      |
    |                 |
    | Connection Pool |
     -----------------
```

Connection kurulması için
- DB Name
- Username
- Password
- Host
- Adapter (driver)

**Model ve Tablo karışılıkları**

```
Model/Class            Table/Schema
Article 	           articles
LineItem 	           line_items
Deer 	               deers
Mouse 	               mice
Person 	               people
```

> 10000000 == 10_000_000

*Primary key*  
*Foreign key*

#### Model'in yazılması

```ruby
class Product < ApplicationRecord
end
```

ApplicationRecord tarafından setter ve getter'ları oluşturur. Bu bir kere oluşturulup cache'lenir.

```ruby
def name #getter method
  @name
end

def name=(name) #setter method
  @name = name
end
```

User.class_methods.add(new_method) gibi birşeyler yapılabiliyor.

`attr_reader` (setter oluşturma)  
`attr_writer` (getter oluşturma)  
`attr_accessor` (setter ve getter oluşturma)  

---

Eğer db'deki farklı bir tabloyu kullanmak, ismini elle girmek istersek

```ruby
class Product < ApplicationRecord
  self.table_name = "my_products"
end
```

şeklinde yapabiliriz.

#### CRUD (Create, Read, Update, Delete)

##### Create

```ruby
user = User.new(name: "Bora", surname: "Tanrikulu")
user.save
```

##### Read

```ruby
# return a collection with all users
users = User.all
```

```ruby
# return the first user
user = User.first
```

```ruby
# return the first user named Bora
david = User.find_by(name: "Bora")
```

```ruby
# find all users named David who are Code Artists and sort by created_at in reverse chronological order
users = User.where(name: 'Bora', surname: 'Tanrikulu').order(created_at: :desc)
```

##### Find

Bir object dönerken, **Where** bir collection dönecektir.

*Collection*, içersinde birden fazla obje saklayan yapıdır.

##### Update

```ruby
user = User.find_by(name: 'Bora')
user.name = 'Asya'
user.save
```

```ruby
user = User.find_by(name: 'Bora')
user.update(name: 'Asya')
```

```ruby
User.update_all "max_login_attempts = 3, must_change_password = 'true'"
```

##### Delete

```ruby
user = User.find_by(name: 'David')
user.destroy
```

```ruby
# find and delete all users named David
User.where(name: 'David').destroy_all
 
# delete all users
User.destroy_all
```

#### Validation

İlgili veriler için kontrol yapabilmemizi sağlar. Bu işlem sunucu tarafı validasyondur. İstemci tarafında da validasyon işlemi yapılabilir (submit butonunu disable yapmak gibi). Fakat asla son kullanıcıya güvenilmemelidir. Sunucu tarafında validasyon yapmak oldukça önemlidir!

```ruby
class User < ApplicationRecord
  validates :name, presence: true
end
 
user = User.new
user.save  # => false
user.save! # => ActiveRecord::RecordInvalid: Validation failed: Name can't be blank
```

```
|
|   İstemci Validasyonu (Javascript, HTML ile vs.)
|
|   Sunucu Controller
|
|   Sunucu Validasyonu - Model Katmanı
|
|   Sunucu Validasyonu - Database Katmano
|
```

#### Callbacks

-

#### Migrations

```ruby
class CreatePublications < ActiveRecord::Migration[5.0]
  def change
    create_table :publications do |t|
      t.string :title
      t.text :description
      t.references :publication_type
      t.integer :publisher_id
      t.string :publisher_type
      t.boolean :single_issue
 
      t.timestamps
    end
    add_index :publications, :publication_type_id
  end
end
```

Migration'ı çalıştırmak:

```ruby
rails db:migrate
```

Migration'ı geri almak:

```ruby
rails db:rollback
```

---

## Migrations

> *Rails-6.0.0-rc1* ile yeni bir proje oluşturduk.

Versiyonları tutmak için database'da schema_table oluşturuyor ve versiyonları orada saklayarak db ile migration'ları eş(sync) olup olmadığını kontrol ediyor.

```sh
rails db:create
```

```sh
rails generate migration CreateProducts name:string description:text
```

```ruby
# db/migrate/create_products.rb
class CreateProducts < ActiveRecord::Migration[6.0]
  def change
    create_table :products do |t|
      t.string :name
      t.text :description
      t.timestamps
    end
  end
end
```

```ruby
rails db:migrate
```

```ruby
# db/schema.rb
ActiveRecord::Schema.define(version: 2019_07_22_151741) do

  create_table "products", force: :cascade do |t|
    t.string "name"
    t.text "description"
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
  end

end
```

```sh
rails generate migration AddPartNumberToProducts part_number:string
```

```ruby
# db/migrate/add_part_number_to_products.rb
class AddPartNumberToProducts < ActiveRecord::Migration[6.0]
  def change
    add_column :products, :part_number, :string
  end
end
```

```ruby
rails db:migrate
```

Migration'larda geri gitmek için *rollback* yaparız.
Aşağıdaki gibi

```sh
ruby db:rollback
```

> Rollback yapmadan migration'ları edit'lemeyin!

Migration'ların durumuna bakmak için:

```ruby
ruby db:migrate:status
```

En son migration'ı aşağıdaki gibi yapıp tekrar migrate çekelim.

```ruby
class AddPartNumberToProducts < ActiveRecord::Migration[6.0]
  def change
    add_column :products, :part_number, :string
    add_index :products, :part_number
  end
end
```

```sh
rails db:migrate
```

**Peki index nedir ?**

Arama sürecini hızlandırmak için index'liyoruz.

index = performans

---

Bir şey silmek için aşağıdaki gibi yapabiliriz

```sh
rails d migration AddDetailsToProducts part_number:string price:decimal
```

```sh
rails db:migrate
```

---

Eğer bir migration'da bir çok şey ekleyecek ise ismi direkt olarak açıklayıcı bir şey yazabiliriz. Ardından elimizle migration dosyasını güncelleriz.

```sh
rails generate migration AddDetailsToProducts part_number:string:index price:decimal:index
```

```ruby
class AddDetailsToProducts < ActiveRecord::Migration[6.0]
  def change
    add_column :products, :part_number, :string
    add_index :products, :part_number
    add_column :products, :price, :decimal
    add_index :products, :price
  end
end
```

```sh
rails db:migrate
```

---

Yeni bir tablo oluşturalım.

```sh
rails generate migration CreateUsers name:string age:integer
```

```ruby
class CreateUsers < ActiveRecord::Migration[6.0]
  def change
    create_table :users do |t|
      t.string :name
      t.integer :age
    end
  end
end
```

#### References

```sh
rails generate migration AddUserRefToProducts user:references
```

User'ların product'ları olsun.

```
    ---------                     ---------
   | Product |  ---------------> | User    |
    ---------                 /   ---------
                             /
    ---------               /
   | Product |  ------ ----/
    --------- 
```

Bu durumda Product altında user_id diye bir column tutarız.

```
user_id: references
```

Yani user_id bir foreign_key(ikincil-yabancı anahtar olacaktır.)

Rails'de yazarken user_id değil user yazarız.

```ruby
class AddUserRefToProducts < ActiveRecord::Migration[6.0]
  def change
    add_reference :products, :user, foreign_key: true
  end
end
```

Reference kolonları yani foreign_key'ler her zaman index'lenir.

#### Join Tables

Ara tablolar.

Many-to-many ilişki

```
    ---------                      ---------
   | Product |  --------------->  | User    |
    ---------                  /   ---------
                              /
    ---------                /
   | Product |  ------------/
    ---------   \ 
                 \
                  ------------------------
                                          \
                                           ---------
                                          | User    |
                                           ---------
```

Customer diye bir tablo yaratalım. Bu sefer artık bunun modelini de oluşturmak istiyoruz. Aşağıdaki gibi oluşturalım. Bu sefer hem model hem de migration oluşturacak.

```sh
rails generate model Customer first_name:string last_name:string email:string
```

```bash
invoke  active_record
create    db/migrate/create_customers.rb
create    app/models/customer.rb
invoke    test_unit
create      test/models/customer_test.rb
create      test/fixtures/customers.yml
```

Model generate ederek yapınca, migration oluştururken timestamp'leri de yazıverdi.

```ruby
class CreateCustomers < ActiveRecord::Migration[6.0]
  def change
    create_table :customers do |t|
      t.string :first_name
      t.string :last_name
      t.string :email

      t.timestamps
    end
  end
end
```

Şimdi product ile customer arasında aratablo oluşturalım.

```sh
rails generate migration CreateJoinTableCustomerProduct customer product
```

```ruby
class CreateJoinTableCustomerProduct < ActiveRecord::Migration[6.0]
  def change
    create_join_table :customers, :products do |t|
      t.index [:customer_id, :product_id]
      t.index [:product_id, :customer_id]
    end
  end
end
```

JoinTable'da yalnızca foreign_key'ler olur. Eğer ek bir şey eklemek istersek(oluşturma tarihi vs. gibi) jointable değil de ayrı bir model olarak yaparız.

```sh
rails db:migrate
```
