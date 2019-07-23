## Validations

Person adından bir model daha oluşturalım.

```sh
rails generate model Person
```

```sh
rails db:migrate
```

Model katmanında gelen veriyi kontrol etmek amacıyla kullanırız.

Dün yaptığımız projeyi ayağa kaldırıp console'a geçelim.

```sh
rails console
```

Bir tane Person objesi oluşturalım

```ruby

```

> İşlemler sırasında sorun çıkarsa
  `rails db:migrate:reset` yapabiliriz

balidates :name presence true

.valid ?  
doğru mu

ona ünlem var ise execption'ları fırlatır

p.error_messages

Şimdi name attribute'ü boş bırakılamaz olarak set'leyelim

**models/person.rb**
```ruby
class Person < ApplicationRecord
  validates :name, presence: true
end
```


error.details[]


---

eğer tektik olarak db'de olmayan ama model'de olanları validate etmek için acceptance kullanılır

```ruby

class Person < ApplicationRecord
  validates :terms_of_service, acceptance: true
end
```

---

Başka bir model ile bağlı olduğunda ve onu validate etmek istediğimizde kullanırız.

```ruby
class Library < ApplicationRecord
  has_many :books
  validates_associated :books
end
```

---

Birşeyi iki defa alıp kontrol ediyorsan confirmation kullanırız.

```ruby
class Person < ApplicationRecord
  validates :email, confirmation: true
end
```

```html
<%= text_field :person, :email %>
<%= text_field :person, :email_confirmation %>
```

---

Exlusion bunlardan herhangi birisi

```ruby
class Account < ApplicationRecord
  validates :subdomain, exclusion: { in: %w(www us ca jp),
    message: "%{value} is reserved." }
end
```

---

format

Elle girdisi alınan verilerin formata uygun olup olmadığını kontrol etmek için kullanılır.

Örnepin telefon numarası girdisi alırken kullanılması gerekebilir.

Regex kullanılarak yapılır.

```ruby
class Product < ApplicationRecord
  validates :legacy_code, format: { with: /\A[a-zA-Z]+\z/,
    message: "only allows letters" }
end
```

---

Bir şeyin uzunluğunu kontrol ederken length kullanırız.

```ruby
class Person < ApplicationRecord
  validates :bio, length: { maximum: 1000,
    too_long: "%{count} characters is the maximum allowed" }
end
```

---

numericality,  

presence,

Bir şeyin tek olmasını istiyorsan uniqueness kullanırız.
```ruby
class Account < ApplicationRecord
  validates :email, uniqueness: true
end
```

Eğer hazır bir validater yok ise kendimizde bir validation class'ı yazabiliriz.

Custom validate'i aşağıdaki gibi yazabiliriz
```ruby
class GoodnessValidator < ActiveModel::Validator
  def validate(record)
    if options[:fields].any?{|field| record.send(field) == "Evil" }
      record.errors[:base] << "This person is evil"
    end
  end
end
```

Bundan sonra aşağıdaki gibi yazdığımız validate'i kullanabiliriz
```ruby
class Person < ApplicationRecord
  validates_with GoodnessValidator, fields: [:first_name, :last_name]
end
```

---

Eğer validate'i farklı action'larda yapacak ise **on**'u kullanabiliriz.
```ruby

class Person < ApplicationRecord
  # it will be possible to update email with a duplicated value
  validates :email, uniqueness: true, on: :create
 
  # it will be possible to create the record with a non-numerical age
  validates :age, numericality: true, on: :update
 
  # the default (validates on both create and update)
  validates :name, presence: true
end
```

---

Validate'e koşul da koyabiliriz.
```ruby
class Order < ApplicationRecord
  validates :card_number, presence: true, if: :paid_with_card?
 
  def paid_with_card?
    payment_type == "card"
  end
end
```

---

Koşullara göre grouplandırma yapılarak validate yapılabilir

```ruby

class User < ApplicationRecord
  with_options if: :is_admin? do |admin|
    admin.validates :password, length: { minimum: 10 }
    admin.validates :email, presence: true
  end
end
```

---

Error'ları string olarak almak için direkt böyle alabiliriz
```ruby
person.errors.messages
```

---

Şimdi öğrendiğimiz bilgiler ışınğında denemeler yapıcaz.

Örneklerimiz için validation_example diye ayrı bir proje oluşturduk.
```sh
rails new validation_example
```

Örneğimiz aşağıdaki gibi olacak.

```
    ------                     ---------
   | USER | <---------------- | PROFILE |
    ------                     ---------
       ^
       |
       |      Profile
       |         - bio (maximum 270 characters)
       |         - age (integer, min:18, max: 99)
       |         - gender (enum: male, female)
       |
       |
    ---------
   | LESSONS |
    ---------


               User
                   - email (uniq, presence)
                   - first_name
                   - last_name
                   - password
                     - confirmation

               Lesson
                   - title
                   - max_person (min:10, max: 40)
```

Modellerimi oluşturualım.

User
```ruby
rails generate model User email:string first_name:string last_name:string password:string
```

Lesson
```ruby
rails generate model Lesson title:string max_person:integer user:references
```

Profile
```ruby
rails generate model Profile bio:text age:integer gender:integer user:references
```

Migration Dosyalarını görüntülüyelim
```sh
cat db/migrate/*
```

```ruby
class CreateUsers < ActiveRecord::Migration[6.0]
  def change
    create_table :users do |t|
      t.string :email
      t.string :first_name
      t.string :last_name
      t.string :password

      t.timestamps
    end
  end
end
class CreateLessons < ActiveRecord::Migration[6.0]
  def change
    create_table :lessons do |t|
      t.string :title
      t.integer :max_person
      t.references :user, null: false, foreign_key: true

      t.timestamps
    end
  end
end
class CreateProfiles < ActiveRecord::Migration[6.0]
  def change
    create_table :profiles do |t|
      t.text :bio
      t.integer :age
      t.integer :gender
      t.references :user, null: false, foreign_key: true

      t.timestamps
    end
  end
end
```

Migration
```sh
rails db:migrate
```

Validation

User
```ruby
class User < ApplicationRecord
  # Associations
  has_one :profile
  has_many :lesson

  # Validations
  validates :email,
            :first_name,
            :last_name,
            :password,
            presence: true

  validates :email,
            uniqueness: true

  validates :password,
            confirmation: true
end
```

Lesson
```ruby
class Lesson < ApplicationRecord
  # Associations
  belongs_to :user

  # Validations
  validates :title,
            presence: true

  validates :max_person,
            numericality: { greater_than_or_equal_to: 10,
                            less_than_or_equal_to: 40 }
end
```

Profile
```ruby
class Profile < ApplicationRecord
  # Associations
  belongs_to :user

  # Enums
  enum gender: { male: 0,
                 female: 1 }

  # Validations
  validates :bio,
            length: { maximum: 250 }

  validates :age,
            numericality: { greater_than_or_equal_to: 18,
                            less_than_or_equal_to: 99 }
end
```
