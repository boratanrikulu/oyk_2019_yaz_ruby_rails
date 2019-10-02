## Polymorphic Associations

Tek bir tablodan birden fazla türde şey tutabilmeyi sağlar.

```
 ---------
| Picture | ---------------|
 ---------                 v
     |                 ---------
     |                | Product |
     v                 ---------
 ----------            
| Employee |
 ----------
```

```ruby
class Picture < ApplicationRecord
  belongs_to :imageable, polymorphic: true
end
 
class Employee < ApplicationRecord
  has_many :pictures, as: :imageable
end
 
class Product < ApplicationRecord
  has_many :pictures, as: :imageable
end
```

Picture'ı hem employee hem de Product için kullanmak istiyoruz. Bu durumda Picture'ı aşağıdaki gibi yaparak Polymorphic yapmalıyız.


```ruby
class CreatePictures < ActiveRecord::Migration[5.0]
  def change
    create_table :pictures do |t|
      t.string  :name
      t.integer :imageable_id
      t.string  :imageable_type
      t.timestamps
    end
 
    add_index :pictures, [:imageable_type, :imageable_id]
  end
end
```

Ya da kısa bir yol olarak
```ruby
class Picture < ApplicationRecord
  belongs_to :imageable, polymorphic: true
end
 
class Employee < ApplicationRecord
  has_many :pictures, as: :imageable
end
 
class Product < ApplicationRecord
  has_many :pictures, as: :imageable
end
```

Başka bir örnek olarak yorum verilebilir. Örneğin sistemimizde Yazar, Kitap, Kitapevi için yorum tutmak istiyoruz bunların hepsi için ayrı birer model oluşturmak yerine tek bir modelde Polymorphic bir yapı kurularak 3 model içinde bir arada tutulabilir. Bunun için tek yapmak gereken model'de commentable_type ve commentable_id tutmaktır.

---

Şimdi örneğimiz için Author ve Book modellerini oluşturalım.
```sh
rails new polymorphic_example
```

```sh
rails generate model Author name:string surname:string
```

```sh
rails generate model Book title:string isbn:int author:belongs_to
```

Şimdi polymorphic bir comment oluşturalım.
```sh
rails generate model Comment body:text commentable:references{polymorphic}
```

```ruby
class CreateComments < ActiveRecord::Migration[6.0]
  def change
    create_table :comments do |t|
      t.text :body
      t.references :commentable, polymorphic: true, index: true

      t.timestamps
    end
  end
end
```

schema'a bakıp her şey yolundamı doğruyalım
```sh
cat db/schema.rb
```

```ruby
ActiveRecord::Schema.define(version: 2019_07_25_080842) do

  create_table "authors", force: :cascade do |t|
    t.string "name"
    t.string "surname"
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
  end

  create_table "books", force: :cascade do |t|
    t.string "title"
    t.integer "isbn"
    t.integer "author_id", null: false
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
    t.index ["author_id"], name: "index_books_on_author_id"
  end

  create_table "comments", force: :cascade do |t|
    t.text "body"
    t.string "commentable_type"
    t.integer "commentable_id"
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
    t.index ["commentable_type", "commentable_id"], name: "index_comments_on_commentable_type_and_commentable_id"
  end

  add_foreign_key "books", "authors"
end
```

Şimdi de model tarafında gerekli bağlantıları kuralım

```ruby
class Author < ApplicationRecord
	has_many :comments, as: :commentable, dependent: :destroy
end
```

```ruby
class Book < ApplicationRecord
  belongs_to :author
  has_many :comments, as: :commentable, dependent: :destroy
end
```

```ruby
class Comment < ApplicationRecord
  belongs_to :commentable, polymorphic: true
end
```

Şimdi test edelim

```sh
rails console
```

```ruby
author = Author.create(name: "George", surname: "Orwell")
```

```ruby
book = author.books.create(title: '1984', isbn: 12351234123)
```

```ruby
book = author.books.create(title: 'Animal Farm', isbn: 51123234123)
```

Author'un kitaplarına bakalım.

```ruby
author.books.pluck(:title, :isbn)
```

Kitaptan yazara erişelim

```ruby
Book.find_by(title: '1984').author
```

Şimdi yorum ekleyelim kitabımıza

```ruby
book = Book.find_by(title: '1984')
```

```ruby
book.comments.create(body: 'Awesome!')
```

```ruby
book.comments.create(body: 'Love it!')
```

```ruby
book.comments.pluck(:title)
```

Author'a yorum ekleyelim.

```ruby
author = Author.find_by(name: 'George')
```

```ruby
author.comments.create(body: 'Good!')
```

```ruby
author.comments.create(body: 'Wow!')
```

Direkt comment üzerinden oluşturalım

```ruby
Comment.create(body: 'woooooow', commentable: author)
```

Yorum üzerinden yorumları çekmek.

```ruby
comment = Comment.first
```

```ruby
comment.commentable
```

---

## Self Joins

Aynı modeli farklı isimlerde kullanabilmemizi sağlar.

```ruby
class Book < ApplicationRecord 
  belongs_to :writer, class_name: "Author", foreign_key: 'author_id'
end
```

```ruby
book.writer
```

En bariz örneği Follow içindir. User hem follower rolünde olacaktır hem de followed.

---

## Trick'ler, Uyarılar

Active Record sorgu atarken cache mekanizmasını kullanır.

```ruby
author.books                 # retrieves books from the database
author.books.size            # uses the cached copy of books
author.books.empty?          # uses the cached copy of books
```

---

Join table oluştururken aşağıdaki gibi oluşturmak yerine

```ruby

class CreateAssembliesPartsJoinTable < ActiveRecord::Migration[5.0]
  def change
    create_table :assemblies_parts, id: false do |t|
      t.integer :assembly_id
      t.integer :part_id
    end
 
    add_index :assemblies_parts, :assembly_id
    add_index :assemblies_parts, :part_id
  end
end
```

Aşağıdaki kısayol kullanılır.

```ruby
class CreateAssembliesPartsJoinTable < ActiveRecord::Migration[5.0]
  def change
    create_join_table :assemblies, :parts do |t|
      t.index :assembly_id
      t.index :part_id
    end
  end
end
```

---

## Namespace

Model'leri namespace'lerin altına alabiliriz.

```
- App
  |---- Author
        |------- Writer
        |------- Reader
```

```ruby
module Author
	class Writer < ApplicationRecord
	end
end
```

```ruby
module Author
	class Reader < ApplicationRecord
	end
end
```

```ruby
Author::Writer
```

```ruby
Author::Reader
```

---

---

Scopes

---

Her zaman dependent: :destroy diyerek tüm çocukları öldürmek istemeyebiliriz. Çocuğu olan herhangi bir şeyi silmemek istersek

```ruby
:restrict_with_exception causes an exception to be raised if there is an associated record
:restrict_with_error causes an error to be added to the owner if there is an associated object
```

Eğer veriyi silmeden yalnızca kullanım dışı yapmak isterisek id'sini nil yapmamız yeterli olacaktır. Bu durumda,
```ruby
:nullify causes the foreign key to be set to NULL. Callbacks are not executed.
```

Destroy'da callback'ler çalışır iken delete'de çalışmaz.

---

author.books -> Collection döner  
book.author -> Object(Model) döner

---

Collection'lara yeni bir eleman eklemek için '<<' kullanılabilir.

```ruby
author.books << Book.create(...)
```

Silmek için

```ruby
author.books.delete(Book.first)
```

---

```ruby
@book_ids = @author.book_ids
```

aynı

```ruby
@book_ids = @author.books.pluck(:id)
```

---

distinct, aynı kayıttan birden fazla var ise bir tane alır

```ruby
class Person
  has_many :readings
  has_many :articles, -> { distinct }, through: :readings
end
```

---

## MyWeeBlog - Analiz

```
# User
Email
Kullanıcı Adı
Parola

# Profil
Tam İsim
İlk İsim
Soy İsim
Yaş

# Post
Body

# Comment -> Post
User

# Like -> Post
User


Dashboard'da blog post'ları listeleyeceğiz. (son yirmi post listelenecek)

Kullanıcı profile sayfası. İsim (30 adet post)
İsme ait olanları listele

Kullanıcı için ufak bir istatistik sayfası. (Toplam beğeni, toplam yorum)
```

Projeyi yaptık. Buradan bakabilirsiniz: [**`https://github.com/ruby-rails-mustafa-akgul-oyyk-2019/rubit`**](https://github.com/ruby-rails-mustafa-akgul-oyyk-2019/rubit)

---

## Query Interface

**Find**

```ruby
post = Post.find(3)
```

ID'ye göre sorgu atıyor.  
Geriye Objeye döndürür.

```ruby
posts = Post.find([1, 10])
```

**Take**

Almak istediğimiz kayıt sayısını belirtiriz.

```ruby
people = Person.all.where(age: 18).take(5)
```

**First**

**Last**

**Order**

```ruby
posts = Post.order(:title).take(5)
```

**Find by**

Obje döner. (first record, eşleşen ilk kayıt)

```ruby
post = Post.find_by(title: '1984')
```

**Find each**

Toplu işlemler için.

```ruby
User.find_each do |user|
  NewsMailer.weekly(user).deliver_now
end
```

İşlem miktar sınırı koymak için

```ruby
User.find_each(batch_size: 5000) do |user|
  NewsMailer.weekly(user).deliver_now
end
```

```ruby
User.find_each(start: 2000) do |user|
  NewsMailer.weekly(user).deliver_now
end
```

```ruby
User.find_each(start: 2000, finish: 10000) do |user|
  NewsMailer.weekly(user).deliver_now
end
```

---

**Where** birden fazla parametreye sorgu atılacak ise

```ruby
Client.where("orders_count = ? AND locked = ?", params[:orders], false)
```

```ruby
Client.where("created_at >= :start_date AND created_at <= :end_date",
  {start_date: params[:start_date], end_date: params[:end_date]})
```

---

**Range Confitions**

```ruby
Client.where(created_at: (Time.now.midnight - 1.day)..Time.now.midnight)
```

---

**Where not**

```ruby
Client.where.not(locked: true)
```

---

**Order**

```ruby
Client.order(created_at: :desc)
# OR
Client.order(created_at: :asc)
```

---

**Limit, Offset**


