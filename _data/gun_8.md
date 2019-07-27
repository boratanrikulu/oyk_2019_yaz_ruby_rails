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

---

# Action Controller

Controller isimleri çoğul.  

```ruby
class ClientsController < ApplicationController
  def new
  end
end
```

```ruby
get '/clients/:status', to: 'clients#index'
```

params[:status]

---

path'in önüne bir şey koymak için

```ruby
def default_url_options
  { locale: I18n.locale }
end
```

### Strong Parameters

Controller'da belirli parametreleri kullanmak içindir. Başka gelir ise işleme almaz. Güvenlik için.

Örneğimize devam edip strong params'ı uygulayalım.

```ruby
class PostsController < ApplicationController
  def index
    @posts = Post.latest_posts
  end

  def show
    @post = Post.find(params[:id])
  end

  def new
    @post = Post.new
  end

  def create
    @post = Post.create(posts_params)
    redirect_to post_path(@post)
  end

  private

    def posts_params
      params.require(:post)
            .permit(:body)
    end
end
```

[https://gist.github.com/peternixey/1978249](https://gist.github.com/peternixey/1978249)

---

# Session

Session server'da, cookie user'da olur.

Default olarak session için oluşturulan cookie'ler user'da tutulur.

Cookie'e 4kb'den fazla boyutta veri yazılmaz.

Eğer daha büyük bir şey tutmak için ActiveRecordStore'da tutulur yani db'de tutulur cookie.

Bir değişken set'lemekten farkı yok

Cookie olarak default olarak oluşturulmuş session cookie ile server'deki session'nı karşılaştırır. Temelde session ve cookie birbirine benzese de farklıdır aslında.

```ruby
class LoginsController < ApplicationController
  # "Create" a login, aka "log the user in"
  def create
    if user = User.authenticate(params[:username], params[:password])
      # Save the user ID in the session so it can be used in
      # subsequent requests
      session[:current_user_id] = user.id
      redirect_to root_url
    end
  end
end
```

Current user var ise döndür, yok ise db'den bulup yolla, session'a yazılan değeri kullanarak.
```ruby
class ApplicationController < ActionController::Base
 
  private
 
  # Finds the User with the ID stored in the session with the key
  # :current_user_id This is a common way to handle user login in
  # a Rails application; logging in sets the session value and
  # logging out removes it.
  def current_user
    @_current_user ||= session[:current_user_id] &&
      User.find_by(id: session[:current_user_id])
  end
end
```

Logout olurken session'lar öldürülür, cookie'ler silinir.

```ruby
class LoginsController < ApplicationController
  # "Delete" a login, aka "log the user out"
  def destroy
    # Remove the user id from the session
    session.delete(:current_user_id)
    # Clear the memoized current user
    @_current_user = nil
    redirect_to root_url
  end
end
```

Bu işlemleri otomatik olarak yapmak için devise diye bir gem var.

[https://github.com/plataformatec/devise](https://github.com/plataformatec/devise)

---

# Flash

flash'ları setlemek için
```ruby
redirect_to root_url, notice: "You have successfully logged out."
redirect_to root_url, alert: "You're stuck here!"
redirect_to root_url, flash: { referral_code: 1234 }
```

flash'ları göstermek için
```ruby
<html>
  <!-- <head/> -->
  <body>
    <% flash.each do |name, msg| -%>
      <%= content_tag :div, msg, class: name %>
    <% end -%>
 
    <!-- more content -->
  </body>
</html>
```

---

# Cookies

```ruby
cookies[:test_cookie] = "cookieeeeeee"
```

```ruby
class CommentsController < ApplicationController
  def new
    # Auto-fill the commenter's name if it has been stored in a cookie
    @comment = Comment.new(author: cookies[:commenter_name])
  end
 
  def create
    @comment = Comment.new(params[:comment])
    if @comment.save
      flash[:notice] = "Thanks for your comment!"
      if params[:remember_name]
        # Remember the commenter's name.
        cookies[:commenter_name] = @comment.author
      else
        # Delete cookie for the commenter's name cookie, if any.
        cookies.delete(:commenter_name)
      end
      redirect_to @comment.article
    else
      render action: "new"
    end
  end
end
```

Cookie'yı şifrelenmiş olarakta tutabiliriz.

```ruby
class CookiesController < ApplicationController
  def set_cookie
    cookies.encrypted[:expiration_date] = Date.tomorrow # => Thu, 20 Mar 2014
    redirect_to action: 'read_cookie'
  end
 
  def read_cookie
    cookies.encrypted[:expiration_date] # => "2014-03-20"
  end
end
```

---

# Render

controllerde xml, json gibi verileri render'layabiliriz.

Gelen isteğe göre aşağıdaki gibi manüpile edebiliriz.

localhost:3000/posts.json gibi adrese gidildiğinde aşağıdaki gibi döneriz.

```ruby
class UsersController < ApplicationController
  def index
    @users = User.all
    respond_to do |format|
      format.html # index.html.erb
      format.xml  { render xml: @posts }
      format.json { render json: @posts }
      format.pdf { render plain: @posts }
    end
  end
end
```

---

Cookie örnekleri.

```ruby
class UsersController < ApplicationController
  def index
  end

  def create_cookie
    session[:name] = params[:name]
    redirect_to users_path
  end

  def delete_cookie
    session[:name] = nil
    redirect_to users_path
  end

  private

    def user_params
      params.require(:user)
            .permit(:email)
    end
end
```

```ruby
<% if session[:name] %>
  <h1>
    <%= "Hey #{cookies[:name]}" %>
  </h1>
  <%= link_to 'Cookie Sil', users_delete_cookie_path, method: :post %>
<% else %>
  <h1>Hadi adını söyle</h1>
  <%= form_with url: users_create_cookie_path, method: :post do |form| %>
    <%= form.label :name %>
    <%= form.text_field :name %>
    <%= form.submit %>
  <% end %>
<% end %>
```

```ruby
post 'users/create_cookie', to: 'users#create_cookie'
post 'users/delete_cookie', to: 'users#delete_cookie'
```

---

Sayı artırmak

```ruby
class UsersController < ApplicationController
  def index
    cookies[:count] = cookies[:count].to_i.next
  end
end
```

Application Layout
```ruby
<%= "Sayaç: #{cookies[:count]}" %>
```

---

# Filters

Filtreler 'before', 'after' ve 'around' olarak çalışabilir.

```ruby
class ApplicationController < ActionController::Base
  before_action :require_login
 
  private
 
  def require_login
    unless logged_in?
      flash[:error] = "You must be logged in to access this section"
      redirect_to new_login_url # halts request cycle
    end
  end
end
```

Örneğin sayaç methodu için bu uygulanabilir.

Sayı artırmak

```ruby
class UsersController < ApplicationController
  before_action :increase_count, only: :index

  def index
  end

  private

    def increase_count
      cookies[:count] = cookies[:count].to_i.next
    end
end
```

**Around kullanımı**

```ruby
class ChangesController < ApplicationController
  around_action :wrap_in_transaction, only: :show
 
  private
 
  def wrap_in_transaction
    ActiveRecord::Base.transaction do
      begin
        yield
      ensure
        raise ActiveRecord::Rollback
      end
    end
  end
end
```

# Request

```ruby
request.host
request.format
request.method
```

# Response

```ruby
response.headers["Content-Type"] = "application/pdf"
```

---

# HTTP Authentications

```ruby
class AdminsController < ApplicationController
  http_basic_authenticate_with name: "humbaba", password: "5baa61e4"
end
```

---

eğer log'a bazı attribute'lerin basılmasını istemiyor isek
```ruby
config.filter_parameters << :password
```

---

Controller'da anlatılan kavramlar
- Action
- Cookie
- Session
- Params
- Strong Params
- Response ve Request objeleri
- Respond_to
- Format
- Before Action vs.
- Redirect to
- Flash

# Routing

```
 ----                                 İSTEK
| DB |                               /
 ----                               /
  ||                        --------
  ||                       | Router |
 -------                    --------
| Model |                /
 -------                /
   \\                  /
    \\                /
     \\   ------------               ------
      \\ | Controller | <---------> | View |
          ------------               ------
```

```
             REST
URL        /posts/new
 |         /posts/index     
 |------------|-----|
        Controller  Action

|___________________________|
      FrontController
          DesignPattern        

```

Router'in yaptığı iç controller'ın ilgili action'nına yönlendirmektir.

Path'e, Method'a, Parametrelere bakarak nereye gidilmesi gerektiğine karar verir.

---

Bir GET isteği yapılmış, bunu yönlendirmeyi sağlar.

```ruby
get '/patients/:id', to: 'patients#show', as: 'patient'
   |----------------|   |---------------|    |----------|
       PATTERN          CONTROLLER/ACTION    patient_path(id)
                                                  |
                                                  v
                                              /patients/:id   
```

as'i yazmaz isek otomatik olarak pattern'den yaratılır.  

```ruby
<%= link_to 'Patient Record', patient_path(@patient) %>
```

Standart, restful dışında bir pattern yazarsanız otomatik olarak helper yaratmaz.

---

Rails Ujs özelliği ile herşeyi post gibi çalıştırır arka planda yakalar

---

**Restful**  
index  
show  
new  
create  
edit  
update  
destroy

---

aaaa_url -> tüm url  (http://a.com:3000/hehehe)  
aaaa_path -> yalnızca path kısmı (/hehehe)

---

Eğer RESTFUL yapacaksak, CRUD yapacaksak direkt olarak resources helper'ını kullanabiliriz.

```ruby
resources :birkelime
```

```
birkelime_index GET    /birkelime(.:format)           birkelime#index
                POST   /birkelime(.:format)           birkelime#create
  new_birkelime GET    /birkelime/new(.:format)       birkelime#new
 edit_birkelime GET    /birkelime/:id/edit(.:format)  birkelime#edit
      birkelime GET    /birkelime/:id(.:format)       birkelime#show
                PATCH  /birkelime/:id(.:format)       birkelime#update
                PUT    /birkelime/:id(.:format)       birkelime#update
                DELETE /birkelime/:id(.:format)       birkelime#destroy
```

only sayesinde istediğimiz action'ların yaratılmasını sağayalabiliriz.

```ruby
resources :birkelime, only: [:index, :show]
```

```
birkelime_index GET    /birkelime(.:format)           birkelime#index
      birkelime GET    /birkelime/:id(.:format)       birkelime#show
```

---

| HTTP  Verb | Path | Controller#Action | Used for |
|:----------:|:----:|:-----------------:|:--------:|
| GET  | /photos | photos#index | display a list of all photos |
| GET  | /photos/new | photos#new | return an HTML form for creating a new photo |
| POST | /photos | photos#create | create a new photo |
| GET  | /photos/:id | photos#show | display a specific photo |
| GET  | /photos/:id/edit | photos#edit | return an HTML form for editing a photo |
| PATCH/PUT |  /photos/:id | photos#update | update a specific photo |
| DELETE | /photos/:id | photos#destroy | delete a specific photo |
