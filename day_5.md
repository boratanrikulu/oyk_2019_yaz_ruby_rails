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

## Callbacks

Hook - Webhook

Life cycle

Event - Event Driven Development

Callback örneği olarak bir kullanıcı üye olduktan sonra e-posta atılması verilebilir.

Trigger (DB), bişey olduğunda tetikle

---

Örneğin aşağıdaki örnekte girilen name ifadesini validate'e sokmadan önce büyük harf yapabiliriz.

```ruby
class User < ApplicationRecord
  validates :login, :email, presence: true
 
  before_create do
    self.name = login.capitalize if name.blank?
  end
end
```

Callback'lerde de validation'da olduğu gibi **on** kullanılarak belli koşullarda kullanabiliriz.

```ruby

class User < ApplicationRecord
  before_validation :normalize_name, on: :create
 
  # :on takes an array as well
  after_validation :set_location, on: [ :create, :update ]
 
  private
    def normalize_name
      self.name = name.downcase.titleize
    end
 
    def set_location
      self.location = LocationService.query(self)
    end
end
```

---

before_validation  
after_validation

```
 >------------------
                    |  Before Validation
                    |  After Validation
                    |  Before Save
                    |  After Save
                    |  Before Commit
                    |  After Commit
 <------------------                   
```

Model işini bitirdikten sonra db'e işlemi yapmak için gider ve commit yapar.

before_commit  
after_save

---

after_initialize  
after_find

```ruby

class User < ApplicationRecord
  after_initialize do |user|
    puts "You have initialized an object!"
  end
 
  after_find do |user|
    puts "You have found an object!"
  end
end
 
>> User.new
You have initialized an object!
=> #<User id: nil>
 
>> User.first
You have found an object!
You have initialized an object!
=> #<User id: 1>
```

---

after_touch ise bir şey update olduğunda tetiklenir.

```ruby

class Employee < ApplicationRecord
  belongs_to :company, touch: true
  after_touch do
    puts 'An Employee was touched'
  end
end
 
class Company < ApplicationRecord
  has_many :employees
  after_touch :log_when_employees_or_company_touched
 
  private
  def log_when_employees_or_company_touched
    puts 'Employee/Company was touched'
  end
end
 
>> @employee = Employee.last
=> #<Employee id: 1, company_id: 1, created_at: "2013-11-25 17:04:22", updated_at: "2013-11-25 17:05:05">
 
# triggers @employee.company.touch
>> @employee.touch
Employee/Company was touched
An Employee was touched
=> true
```

---

Aşağıdaki durumlarda callback yapılamaz.

decrement  
decrement_counter  
delete  
delete_all  
increment  
increment_counter  
toggle  
update_column  
update_columns  
update_all  
update_counters

Destroy'da callback fırlar iken delete'de fırlamaz..

---

`throw :abort` yapınca lifecycle geri sarıp başa dönebiliriz.

---

Parent'ler silindiğinde child'ları da silmek istersek
```ruby
class User < ApplicationRecord
  has_many :articles, dependent: :destroy
end
```

---

Aynı callback birden fazla yerde yapılıyor ise ortak callback'ler app/callbacks dizi içersinde bir class olarak oluşturularak ortakça kullanılabilir.

---

DB'deye kesinlikle yazıldıktan sonra bi işlem yapıyorsak

`after_commit` ile yaparız.

---

Şimdi öğrendiğimiz bilgiler ile projemize devam edelim.

Projede User'a full_name eklicez ve bu full_name, first ve last name'ler ile dolacak. Ve zorunlu.

```sh
rails generate migration AddFullNameToUsers full_name:string
```

```ruby
# Validations
validates :email,
          :full_name
          :first_name,
          :last_name
          :password,
          presence: true
```

```ruby
# Callbacks
before_validation :prepare_full_name

private

def prepare_full_name
  self.full_name = "#{first_name} #{last_name}"
end
```

---

## Associations

> Rails'i rails yapan işlev.

Model'ler arasında ilişki ağı kurmak için (database'deki gibi).

Veritabanımız ilişkisel, modellerimizde ilişkisel yapıyoruz.

Öksüz kayıt durumunu çözmek için ilişki kurulurken `dependent: :destroy` kullanılır.

Örneğin yazarın kitapları olsun, yazar silinirse kitaplar da silinecektir.

```ruby
class Author < ApplicationRecord
  has_many :books, dependent: :destroy
end
 
class Book < ApplicationRecord
  belongs_to :author
end
```

@author.destroy

@author.books'larda silinecektir.

---

Author has many books.  
Book belongs to author.  

Author has one book.  
Book belongs to author.

---

```ruby
@book = @author.books.create(published_at: Time.now)
```

Yazarın kitaplarına yeni bir kitap oluştur.

Bu şekilde ilişki kurmasaydık aşağıdaki gibi yazardık, yukardaki çok daha hoş.
```ruby
@book = Book.create(published_at: Time.now, author_id: @author.id)
```

---

- belongs_to (ait)
- has_one (bir tane sahip)
- has_many (bir sürü var)
- has_many :through (bir sürü sahip ama bişey üzerinden)
- has_one :through (bi tane sahip ama bişey üzerinden)
- has_and_belongs_to_many (many-to-many)

---

## belongs_to

```ruby
class Book < ApplicationRecord
  belongs_to :author
end
```

```
 -------      belongs_to          ---------
| books | ---------------------> | authors |
 -------                          ---------
 author_id
```

## has_one

```ruby
class Supplier < ApplicationRecord
  has_one :account
end
```

```
 -----------                    ----------
| suppliers | <--------------- | accounts |
 -----------                    ----------
 has_one :account               supplier_id
                                belongs_to :supplier
```

## has_many

```ruby
class Author < ApplicationRecord
  has_many :books
end
```

```
 -------                          ---------
| books | ---------------------> | authors |
 -------                          ---------
 author_id                       has_many :books
```

## has_many :through

```ruby
class Physician < ApplicationRecord
  has_many :appointments
  has_many :patients, through: :appointments
end
 
class Appointment < ApplicationRecord
  belongs_to :physician
  belongs_to :patient
end
 
class Patient < ApplicationRecord
  has_many :appointments
  has_many :physicians, through: :appointments
end
```

```
 -----------
| physician | <-----------|
 -----------              |            --------------
                          |-----------|              |
                                      | Appointments |
                          |-----------|              |
 -----------              |            --------------
| patients  | <-----------|
 -----------
```

```ruby
class CreateAppointments < ActiveRecord::Migration[5.0]
  def change
    create_table :physicians do |t|
      t.string :name
      t.timestamps
    end
 
    create_table :patients do |t|
      t.string :name
      t.timestamps
    end
 
    create_table :appointments do |t|
      t.belongs_to :physician, index: true
      t.belongs_to :patient, index: true
      t.datetime :appointment_date
      t.timestamps
    end
  end
end
```

---

Farklı bir örnek

```ruby
class Document < ApplicationRecord
  has_many :sections
  has_many :paragraphs, through: :sections
end
 
class Section < ApplicationRecord
  belongs_to :document
  has_many :paragraphs
end
 
class Paragraph < ApplicationRecord
  belongs_to :section
  has_one :document, through: :section
end
```

---

## has_one :through

```ruby
class Supplier < ApplicationRecord
  has_one :account
  has_one :account_history, through: :account
end
 
class Account < ApplicationRecord
  belongs_to :supplier
  has_one :account_history
end
 
class AccountHistory < ApplicationRecord
  belongs_to :account
end
```

```
 ----------
| supplier | <------------|
 ----------               |            ---------
                          |-----------|         |
                                      | account |
                                 |--->|         |
 ------------------              |     ---------
| account_history  | ------------|
 ------------------
```

## has_and_belongs_to_many

JoinTable

```ruby
class Assembly < ApplicationRecord
  has_and_belongs_to_many :parts
end
 
class Part < ApplicationRecord
  has_and_belongs_to_many :assemblies
end
```

## has_one ve belongs_to farkı

```ruby
class Supplier < ApplicationRecord
  has_one :account
end
 
class Account < ApplicationRecord
  belongs_to :supplier
end
```
