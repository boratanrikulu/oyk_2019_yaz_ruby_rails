# Ruby Gems

> [rubygems.org/](https://rubygems.org/)

- S3 altında saklanıyor tüm gem'ler.
- Açık kaynak.
- Bize bir api verir. gem install filan deyince o api'den çekiyoruz. (source ile veriyoruz)
- https://rubygems.org/stats

- https://github.com/erikhuda/thor

gem programında kullanabileceğimiz parametreler  
https://guides.rubygems.org/command-reference/

---

# Ruby

[https://ruby.github.io/TryRuby/](https://ruby.github.io/TryRuby/)

```ruby
class HelloWorld
	def initialize(name)
		@name = name
	end

	def say_hey
		puts "Merhaba #{@name}"
	end
end

hello_world = HelloWorld.new('Dünya')
hello_world.say_hey
```

---

`gets ve gets.chomp` birbirinden farkı; gets.chomp yapıldığında girdinin sonundaki `\n` silinir.

---

Üst class'ları görmek için `superclass` kullanılabilir.

```ruby
1.class.superclass.superclass.superclass
```

---

```ruby
'bora'.to_i.class
'bora'.to_i
```

---

Ruby'de soru işareti olan method'lar her zaman boolean döner. True/false dönen method'lara soru işareti eklemeliyiz.

---

Bir class'mı diye bakmak
```ruby
3.15.is_a?(Integer)
3.15.integer?

5.even?
5.odd?
```

---

$global_değişken -> Her yerden erişilir  
@instance_variable -> Class'ın örnek değişkeni  
@@class_variable

SABIT_SAYI = 3.3

---

Değişkenin değişmesini istemediğimizde symbol kullanıyoruz. object_id hep aynı.

```ruby
:semboooool.object_id
```

---

```ruby
%w(1,2,3)
````

---

Array'e eleman eklemek için `<<` kullanmak önerilir. En sona ekler.

İki eleman arasına eklemek için `insert` önerilir.

---

#### Enums

Enumrator, sayılabilir hale getiriyor.

Enumrable

```ruby
Enumrable.instance_methods
```

map! yapınca kalıcı hale geliyor.

collect, map'in alias'ı

`z ||= 3`, eğer z boş ise değeri ata


---

#### Methods

```ruby
def method_name(param1, param2=3)
	# Do something
end
```

Multi params. Birden fazla parametre alabileceğimizde kullanırız.

```ruby
def method_name(*multiparam)
	puts multiparam
end
```

---

Ruby'de getter ve setter method'larını kolay yoldan tanımlamak,
attr_reader, attr_writer, attr_accessor,

https://mixandgo.com/learn/ruby_attr_accessor_attr_reader_attr_writer

---

self üzerinden yaratılan method'lar **static** olur.

örneğin

```ruby
def self.finder_numbers
	10
end
```

bu method'u objeyi yaratmadan kullanabiliriz.

---

Ruby Style Guide, https://github.com/rubocop-hq/ruby-style-guide

Hiç bir zaman set, get gibi isimler konulmamalı,
[rubocop](https://github.com/rubocop-hq/ruby-style-guide#accessor_mutator_method_names)

boolean method'lara is, can, do koymamak lazım

[rubocooooop](https://github.com/rubocop-hq/ruby-style-guide#bool-methods-prefix)

---

#### Kalıtım (inheritance)

module include yapılınca method'ları kullanılabilir olur.

---

# Ruby Gem Yazımı

Şimdi bir gem yazıcaz. Amacımız gelen ifadeye göre hangi plaka numarasını söylemek.

```ruby
bundle gem license_plate_parser
```

Gemspec dosyası config dosyası gibidir.
```ruby
gem build license_plate_parser.gemspec
```

değişiklik yaptıkan sonra version değiştirmeden test edebilmek amacıyla
```ruby
pry -I lib -r license_plate_parser
```

```ruby
require 'license_plate_parser'
```

```ruby
LicensePlateParser::Plate.hello
```
