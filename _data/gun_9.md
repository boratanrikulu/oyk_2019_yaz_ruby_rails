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
