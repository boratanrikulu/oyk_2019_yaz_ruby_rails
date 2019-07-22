* cloudflare

request pdf için gelir ise, acceptlanguage pdf olarak, html set'lenir ise text/html olarak geri dönül yapılır.

## Rack

https://rack.github.io/

Ruby'nin http isteklerini anlayabilmesi için araya Rack katmanı girer.

`Rack provides a minimal interface between webservers that support Ruby and Ruby frameworks.`

Herhangi bir dizinde config.ru var ise o bir webserver'dır. Gelen isteklere cevap verebilir.

proje ayaklandığında puma adında bir web sunucusu çalışır.

puma (application web server) (http://puma.io/)
thin ruby webserver
ruby unicorn

config.ru
```
require './myapp.rb'
run myapp
```

myapp.rb
```
require 'rack'
require "rack/handler/puma"
 
myapp = Proc.new do |env|
    ['200', {'Content-Type' => 'text/html'}, ['A barebones rack app.']]
end
 
Rack::Handler::Puma.run myapp
```

```
rackup
```

Development'dayken app server, Production'da ise webserver kullanılır direkt olarak app server kullanılır.

Nginx statik veriler, ssl, saldırılardan sorumlu olarak ayarlanır genelde, puma ve alternatif app serverlar ise ruby kodları çalıştırmak için vs.

Nginx vbir proxy görevi görmüş olur aslında.

CI

Session, cookie. Sunucunun client'ı tanımasını sağlar.

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
 | Model |                | View |
    \                         /
     \                       /
      \                     /
       \                   /
        | Controller       |
        | Routing          |
        | Front Controller |
```

```
- Design Pattern
	- FrontController
		- Routing   <----------------| Dispatch
		- Controller Classes --------|
		- URI
```

FrontController adında design pattern var. MVC'lerin hepsi başlangıçta FrontController uygulayarak başlar. Bu routing ve controller class'larının ilişkili olarak çalışmasını sağlar.

**HTTP Request Methods** [REST ile doğrudan ilgisi var!]
```
GET
POST
PUT/PATH
DELETE
```

FrontController root path'den sonraki alt path'leri algılar ve nereye yani hangi controller'a istek yapılması gerektiğini anlar yani dispatch yapar!

Yapılan http request tipine göre de controller'ın hangi method'u yani hangi action'ı çalışacağı anlaşılır.

Controller'a gidilten sonra eğer DB üzerinde bir işlem yapılacak ise Model katmanı araya girer ve DB üzerinde ilgili sorgular yapılır. (SQL)

SQL sorguları iki gruba ayrılır. SQL aslında bir DSL'dir yani Domain Specific Language.

**DFL** - Data Defination Language
**DML** - Data Manupilation Lanugage

Rails'de SQL sorguları model üzerinde otomatik olarak oluşturulur. Bunu ORM sağlar yani Object Relation Mapping.

---

Model katmanında yapılacak sorgu işlemleri yapıldıktan sonra controller view katmanınına gider.

**View**

- assets
- layout
- template
- render

controllers/users_controller.rb  
views/users/index.html.erb  
models/user.rb


