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

Cache'nin nereye yazılacağını seçmek için
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

Log'ların format'ını belirtmek için
```ruby
config.log_formatter
```

Log'lama seviyesi belirtmek için. Defaul olarak tüm env'lerde debugging'dir.
```ruby
config.log_level
```
