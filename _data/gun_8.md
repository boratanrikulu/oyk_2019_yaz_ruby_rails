# Form Helpers

5.1 ile birlikte form_with geldi. Form_for diye bir şey de var. Form_with default olarak ajax atabilecek şekilde çalışır.

Form helper'ı test edebilmek için rubit örneğimizde posts#new action'nı oluşturalım

posts_controller
```ruby
def new
end
```

routes
```ruby
get 'posts/new'
```

Ardından view dosyamızı yaratalım.  
posts/new.html.erb  
```ruby
<% content_for :action_name,
               render('layouts/shared/current_action',
                      parent_page: 'new')
%>
<%= content_for :page_title, 'Posts details' %>

<h1>Posts#new</h1>
<p>Find me in app/views/posts/new.html.erb</p>
```

Şimdi form_with helper ile bir form oluşturalım.
```ruby
<%= form_with do %>
  Form contents
<% end %>
```

Güvenlik açısından bazı key'ler kullanılır. 
[https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF))

url ve method parametreleri ile aşağıdaki gibi yapabiliriz.
```ruby
<%= form_with(url: "/search", method: "get") do %>
  <%= label_tag(:q, "Search for:") %>
  <%= text_field_tag(:q) %>
  <%= submit_tag("Search") %>
<% end %>
```
Submit yapıldığında aldığı parametreleri belirlenen hedefe belirlenen method şeklinde yollar.

---

## Form Helpers

```ruby
<%= text_area_tag(:message, "Hi, nice site", size: "24x6") %>
<%= password_field_tag(:password) %>
<%= hidden_field_tag(:parent_id, "5") %>
<%= search_field(:user, :name) %>
<%= telephone_field(:user, :phone) %>
<%= date_field(:user, :born_on) %>
<%= datetime_local_field(:user, :graduation_day) %>
<%= month_field(:user, :birthday_month) %>
<%= week_field(:user, :birthday_week) %>
<%= url_field(:user, :homepage) %>
<%= email_field(:user, :address) %>
<%= color_field(:user, :favorite_color) %>
<%= time_field(:task, :started_at) %>
<%= number_field(:product, :price, in: 1.0..20.0, step: 0.5) %>
<%= range_field(:product, :discount, in: 1..100) %>
```

Bu helper'ın hepsi tüm tarayıcılar tarafından desteklenmeyebilir. Kullanırken test edilmesi gerekebilir. Hangi tarayıcılar destekliyor görmek için: [https://modernizr.com/](https://modernizr.com/)

---

posts_controller
```ruby
  def new
    @post = Post.new
  end
```

Yaparak form'da da modeli belirterek o model için yapılması sağlayabiliriz.

Routes
```ruby
Rails.application.routes.draw do
  resources :posts
end
```

Form
```ruby
<%= form_with(model: @post) do %>
  <%= submit_tag("Search") %>
<% end %>
```

Artık modele göre ne yapmak istediğimiz otomatik anlayacak. Örneğin biz boş bir post oluşturmuştuk, bu yazmak istediğimizi anlayarak create action'a post isteği atacaktır.

create action'da oluşturalım.
```ruby
def create
  render json:params
end
```

Default olarak data-remote:true olduğu ajax isteği atar. Bunu tarayıcımızda network'te XHR etiketi altında görebiliriz. Ajax isteği atmaması için local:true olarak setlemeliyiz.

Form
```ruby
<%= form_with(model: @post, local: true) do %>
  <%= submit_tag("Search") %>
<% end %>
```

Peki create etmek istediğimizi nasıl anladı ? Çünkü içi boş. Eğer içi dolu olsaydı update etmesi gerektiğini bilecekti.

Form'un girdi değerlerini de model etkiketi içinde çalışacak şekilde yapalım.

Form
```ruby
<%= form_with model: @post, local: true do |form| %>
  <%= form.text_field :title %>
  <%= form.submit "Search" %>
<% end %>
```

---

İstersek model kullanırken url, path vs verebiliriz.

---

Form_with kullanırken namespace'de kullanabiliriz. Örneğin admin'nin içinde post'a yapmak istiyorsak.

```ruby
form_with model: [:admin, @post]
```

Browser'lar aslında sadece get ve post'u tanır. Diğerleri için ekstra bir input alanı eklenerek method oraya yazılır.  
REST.

---

File Upload için ActiveStroge

---

Simple_form

herşey onun için input olarak yazılıyor, o tipine bakarak hepsini anlıyor.

[https://github.com/plataformatec/simple_form](https://github.com/plataformatec/simple_form)

Rails'deki keskin bıçaklar herşeyi yapmanıza olanak sağlıyor.

---

## Nested Forms

`accepts_nested_attributes_for`

Form içinde form yapabilmemizi sağlar. 

Örneğin tek bir form'da hem user hem de profil bilgileri alıyor olalım. 

User model'inde
```ruby
accepts_nested_attributes_for :profile
```

Artık rails otomatik olarak user formu içinde profil bilgileri gelebileceğini bilecek.

```ruby
<%= form_with model: @user do |f| %>
  <%= f.text_field :name %>
  Profile:
  <ul>
    <%= f.fields_for :profile do |addresses_form| %>
      <li>
        <%= addresses_form.label :first_name %>
        <%= addresses_form.text_field :first_name %>
 
        <%= addresses_form.label :last_name %>
        <%= addresses_form.text_field :last_name %>
        ...
      </li>
    <% end %>
  </ul>
<% end %>
```

Artık gelen parametreleri kullanarak direkt yaratabiliriz.

```ruby
def create
  @user = User.new(user_params)
  # ...
end
 
private
  def user_params
    params.require(:user).permit(:name, profile_attributes: [:first_name, :last_name])
  end
```
