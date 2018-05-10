**Ruby on Rails Guides (v5.2.0) - Getting Started with Rails**
==============================================================

- see [http://guides.rubyonrails.org/](http://guides.rubyonrails.org/)

## 3 Creating a New Rails Project

### 3.1 Installing Rails

- install using Homebrew: `brew install ruby`
- you need to restart terminal so that new ruby version is in your path
- from what I have seen, it creates an alias in `/usr/local/bin/` to ruby executable
- in RubyMine, when asked for SKD, point to the above mentioned `/usr/local/bin/ruby` file
- `sqlite3 --version` already works on my mac
- install Rails globally (from any folder): `gem install rails`

### 3.2 Creating the Blog Application

- execute `rails new blog`
- in `blog` folder execute: `bundle install`


- this is what is created:
  - `app/` - contains controllers, models, views, helpers, mailers, channels, jobs and assets
  - `bin/` - contains the rails scripts used to start you app, and other scripts used to setup, update, deploy and run your application
  - `config/` - configuration for routes, database etc.
  - `config.ru` - Rack configuration for Rack based servers used to start the application
  - `db/` - contains current database schema and migrations
  - `Gemfile`, `Gemfile.lock` - here you can specify what Gem dependencies are needed for your Rails application; used by Bundler gem
  - `lib/` - extended modules for your application
  - `log/` - application log files
  - `package.json` - defines npm dependencies for your application; used by Yarn
  - `public` - the only folder deployed as-is; contains static files and compiled assets
  - `Rakefile` - locates and loads tasks that can be run from the command line; rather than modifying this file, you should add your own tasks to `lib/tasks`
  - `README.md` - brief introduction to your app, you should write this yourself
  - `test/` - unit tests, fixtures etc.
  - `tmp/` - temporary files (like cache and pid files)
  - `vendor/` - a place for all third-party code, which includes vendored gems
  - `.gitignore` - git ignore file
  - `.ruby-version` - contains default ruby version
  
## 4 "Hello", Rails!

### 4.1 Starting up the Web Server

- in `blog` directory execute: `bin/rails server`
- you should be able to access the server at `localhost:3000`

### 4.2 Say "Hello", Rails

- generate controller: `bin/rails generate controller Welcome index`
- it does the following:


    create app/controllers/welcome_controller.rb
     route get 'welcome/index'
    invoke erb
    create   app/views/welcome
    create   app/views/welcome/index.html.erb
    invoke test_unit
    create   test/controllers/welcome_controller_test.rb
    invoke helper
    create   app/helpers/welcome_helper.rb
    invoke   test_unit
    invoke assets
    invoke   coffee
    create     app/assets/javascripts/welcome.coffee
    incoke   scss
    create     app/assets/stylesheets/welcome.scss
    
- to set the root route, add `root welcome#index` to `config/routes.rb`

## 5 Getting Up and Running

- resource in Rails is essentially represents a database table; it will have CRUD operations attached
- change `config/routes` to this to add routes for `articles`:


    Rails.application.routes.draw do
      get 'welcome/index'
       
      resources :articles
       
      root 'welcome#index'
    end
    
- then use `bin/rails routes` to see the actual routes that we have
- it displays the following:


           Prefix Verb URI Pattern                  Controller#Action
    welcome_index GET    /welcome/index(.:format)     welcome#index
         articles GET    /articles(.:format)          articles#index
                  POST   /articles(.:format)          articles#create
      new_article GET    /articles/new(.:format)      articles#new
     edit_article GET    /articles/:id/edit(.:format) articles#edit
          article GET    /articles/:id(.:format)      articles#show
                  PATCH  /articles/:id(.:format)      articles#update
                  PUT    /articles/:id(.:format)      articles#update
                  DELETE /articles/:id(.:format)      articles#destroy
             root GET    /welcome/index(.:format)     welcome#index
    // the rest is some rails stuff
           rails_service_blob GET    /rails/active_storage/blobs/:signed_id/*filename(.:format)                               active_storage/blobs#show
    rails_blob_representation GET    /rails/active_storage/representations/:signed_blob_id/:variation_key/*filename(.:format) active_storage/representations#show
           rails_disk_service GET    /rails/active_storage/disk/:encoded_key/*filename(.:format)                              active_storage/disk#show
    update_rails_disk_service PUT    /rails/active_storage/disk/:encoded_token(.:format)                                      active_storage/disk#update
         rails_direct_uploads POST   /rails/active_storage/direct_uploads(.:format)                                           active_storage/direct_uploads#create
         
         
- note on above:
  - a `resources` in `routes.rb` seems to create 8 route/method combinations
  - `articles GET` - returns a view of all items (index)
  - `articles POST` - creates a new item (create)
  - `new_article GET` - returns a view for adding a new item (new)
  - `edit_article GET` - returns a view for editing an item with `:id` (edit)
  - `article GET` - return a view for viewing an item with `:id` (show)
  - `article PATCH/PUT` - updates an item with `:id` (update)
  - `article DELETE` - deletes an item with `:id`
  
  
- trying to go to `http://localhost:3000/articles/new` now will cause a routing error
         
### 5.1 Laying down the groundwork

- generate articles controller: `bin/rails generate controller Articles`
- trying to go to `http://localhost:3000/articles/new` now will cause an 'unknown action' error


- adding `new` action:


    class ArticlesController < ApplicationController
      def new
      end
    end
    
- trying to go to `http://localhost:3000/articles/new` now will cause `ActionController::UnknownFormat in ArticlesController#new`
  - it will first try to find a template in `articles/new`, then in `application/new`
  - the error message mentions `request.formats: ["text/html"]` which specifies the format the template is to be served in the response; it is by default `text/html` because we requested it using a browser
  - `request.variants: []` specifies what kind of physical devices would be used to see the response and helps Rails determine which template to use; empty here because no information was provided


- to fix the above, create view `app/views/articles/new.html.erb`
- the first extension of the file `new.html.erb` (`html`) represents the template type, while the second one (`erb`) represents the handler that will be used to render it


- `http://localhost:3000/articles/new` now works

### 5.2 The first form

- enter this into `articles/new.html.erb`:


    <%= form_with scope: :article, local: true do |form| %>
      <p>
        <%= form.label :title %><br>
        <%= form.text_field :title %>
      </p>
      <p>
        <%= form.label :text %><br>
        <%= form.text_area :text %>
      </p>
      <p>
        <%= form.submit %>
      </p>
    <% end %>
    
- by default `submit` will want to POST on the same url we are currently on, so `articles/new`
- since there is no such POST route, we will get a `Routing Error` if we try to submit now
- we need to use `POST /articles(.:format) articles#create` route
- so we change the first line to: `<%= form_with scope: :article, url: articles_path, local: true do |form| %>`
- `articles_path` helper tells Rails to point to the path associated with `articles` prefix; also, by default form submit will use POST
- the `*_path` helpers, like in the line above seems to have the `<route_prefix>_path` format, where `<route_prefix>` can be seen by listing routes (`bin/rails routes`)
- now if we try to submit, we will get `Unknown action: The action 'create' could not be found for ArticlesController`
- also, by default `form_with` submits using Ajax and therefore skips full page redirects; we disabled that for now with `local: true`

### 5.3 Creating Articles

- we can now create an empty `create` method in `ArticlesController`
- if we now try to submit, nothing will happen (we will get `204 No Content` HTTP response) - this is because `create` does nothing
- `create` should save the post into the database
- when form is submitted, form fields are sent to Rails as parameters
- we can see them like this:


    def create
      render plain: params[:article].inspect
    end

- we get something like this: `<ActionController::Parameters {"title"=>"First Article!", "text"=>"This is my first article."} permitted: false>`

### 5.4 Creating the Article model

- to create a new model, execute: `bin/rails generate model Article title:string text:text`
- the above creates something like this:


    invoke active_record
    create   db/migrate/20180507120146_create_articles.rb
    create   app/models/article.rb
    invoke   test_unit
    create     test/models/article_test.rb
    create     test/fixtures/articles.yml
    
### 5.5 Running a Migration

- the above `bin/rails generate model` crated a database migration
- you can run migrations and undo them
- the migration file names have timestamps which ensure that migrations are run in order
- execute `bin/rails db:migrate` to migrate
- since I am working in a development environment by default, this migration is applied to the database define in the `development` section of `config/database.yml` (this can be for example `db/development.sqlite3`)
- to execute migration in another environment you must explicitly pass it when invoking migration: `bin/rails db:migrate EAILS_ENV=production`

### 5.6 Saving data in the controller

- update `create` action in `ArticlesController`


    def create
      @article = Article.new(params[:article])
      @article.save
      redirect_to @article
    end
    
- trying to create a new article will still not work, we get `ActiveModel::ForbiddenAttributesError in ArticlesController#create`
- for security reasons, Rails requires us to whitelist allowed controller parameters


    @article = Article.new(params.require(:article).permit(:title, :text))
    
- this is often refactored out to a new method so it can be reused

    
    def create
      @article = Article.new(article_params)
      @article.save
      redirect_to @article
    end
     
    private
    
    def article_params
      params.require(:article).permit(:title, :text)
    end
    
- finally adding an article has succeeded, but there is no controller action yet for the route we have been redirected to; we get `Unknown action: The action 'show' could not be found for ArticlesController`

### 5.7 Showing Articles

- we need to add action for this route: `article GET /articles/:id(.:format) articles#show`
- a frequent practice is to order the standard CRUD operations like this: `index`, `show`, `new`, `edit`, `create`, `update`, `destroy`
- here is the `show` action:


    def show
      @article = Article.find(params[:id])
    end
    
- the above will find an article based on the input parameter `:id` which is specified in the url path, and store it in an instance variable, `@article`
- Rails will automatically pass all instance variables to the view


- now we must create `app/views/article/show.html.erb`


    <p>
      <strong>Title:</strong>
      <%= @article.title %>
    </p>
    <p>
      <strong>Text:</strong>
      <%= @article.text %>
    </p>
    
### 5.8 Listing all articles

- we need to implement the list of articles: `articles GET /articles(.:format) articles#index`


    def index
      @articles = Article.all
    end
    

- view is located at `app/views/article/index.html.erb`:


    <h1>Listing articles</h1>
     
    <table>
      <tr>
        <th>Title</th>
        <th>Text</th>
        <th></th>
      </tr>
       
      <% @articles.each do |article| %>
        <tr>
          <td><%= article.title %></td>
          <td><%= article.text %></td>
          <td><%= link_to 'Show', article_path(article) %></td>
        </tr>
      <% end %>
    </table>

### 5.9 Adding links

- we can add the following to `welcome/index.html.erb`: `<%= link_to 'My Blog', controller: 'articles' %>`
- `link_to` is one of Rails' built-in view helpers - in the above case we create a link to articles
- to `articles/index` view: `<%= link_to 'New article', new_article_path =>`
- to `articles/new` view: `<%= link_to 'Back', articles_path %>`
- to `articles/show` view: `<%= link_to 'Back', articles_path %>`


- if you want to link to an action in the same controller, you don't need to specify the `:controller` option, Rails will use current controller by default
- in the development mode (the default one), Rails reloads the application on every server request, so there is no need to restart the server when change is made

### 5.10 Adding Some Validation

- `app/models/article.rb` currently contains only an 'empty' class so far
- `Article` inherits from `ApplicatonRecord` which inherits from `ActiveRecord::Base`
- `ActiveRecord::Base` supplies a great deal of functionality, including basic CRUD operations, data validation, search and model relationship support


- implement title validation:


    class Article < ApplicationRecord
      validates :title, presence: true, length: { minimum: 5 }
    end

- the above ensures that all articles have a title, and that it is at least 5 characters long
- with validation, `@article.save` will return false on unsuccessful validation
- we also need to set an `@article` in `new`


    def new
      @article = Articles.new
    end
     
    def create
      @article = Article.new(article_params)
       
      if @article.save
        redirect_to @article
      else
        render 'new'
      end
    end
    
- if the validation fails, we need to show the form again
- that is not sufficient, we also need to display some error information
- add the following `articles/new.html.erb` into the form, as the first element inside it:


    <% if @article.errors.any? %>
      <div id="error_explanation">
        <h2>
          <%= pluralize(@article.errors.count, "error") %> prohibited this article from being saved:
        </h2>
        <ul>
          <% @article.errors.full_messages.each do |msg| %>
            <li><%= msg %></li>
          <% end %>
        </ul>
      </div>
    <% end %>
    
- we needed to add `@article = Article.new` to `new` action in controller, because otherwise `@article` would be `nil` in the view and `@article.errors.any?` would throw and exception
- Rails will automatically wrap a field with errors in a `<div class="field_with_errors"></div>` which can be used for styling

### 5.11 Updating Articles

- create `edit` action which simply finds an article by `:id`


    def edit
      @articles = Article.find(params[:id])
    end
    
- create the `articles/edit.html.erb` view that is almost exactly the same as `articles/new` view, except for the title and `form_with` line:


    <h1>Edit article</h1>
     
    <%= form_with(model: @article, local: true) do |form| %>

- passing the `@article` model automagically creates url for submitting the edited article form
- this option tells Rails to use PATCH method which you are expected to use to update resources
- `model: @article` fills the form with the values from `@article`, while `scope: :article` used in `new` view simply creates the fields without putting anything into them


- here is the `update` action:


    def update
      @article = Article.find(params[:id])
     
      if @article.update(article_params)
        redirect_to @article
      else
        render 'edit'
      end
    end

- it is not necessary to pass all the attributes, you can for example update only `title`: `@article.update(title: 'A new title')`


- finally, we need to add `edit` link to `articles/index` view:


    <td><%= link_to 'Edit', edit_article_path(article) %></td>
    
### 5.12 Using partials to clean up duplication in views

- our `edit` and `new` pages are very similar, we can use partials to extract duplicated code
- by convention, partial files are prefixed with an underscore
- create `articles/_form.html.erb`, copy everything from `edit` except title and link
- we can change `new` view (that is `form_with` part) to have the same parameters as `edit`, and continuing on that, use the above partial for both:


    <h1>New article</h1>
    <%= render 'form' %>
    <%= link_to 'Back', articles_path %>
    
<!---->

    <h1>Edit article</h1>
    <%= render 'form' %>
    <%= link_to 'Back', articles_path %>
    
### 5.13 Deleting Articles

- delete route is: `DELETE /articles/:id(.:format) articles#destroy`


    def destroy
      @article = Article.find(params[:id])
      @article.destroy
      redirect_to articles_path
    end
    
- also add the delete link to `articles/index`


    <td><%= link_to 'Destroy', article_path(article), method: :delete, data: { confirm: 'Are you sure?' } %></td>
    
- confirm dialog logic is defined in `rails-ujs` javascript file which is automatically included in `app/views/layouts/application.html.erb`

## 6 Adding a Second Model

### 6.1 Generating a Model

- execute: `bin/rails generate model Comment commenter:string body:text article:references`


    invoke active_record
    create   db/migrate/20140120201010_create_comments.rb
    create   app/models/comment.rb
    invoke   test_unit
    create     test/models/comment_test.rb
    create     test/fixtures/comments.yml
    
- check out `models/comment.rb`:


    class Comment < ApplicationRecord
      belongs_to :article
    end
    
- `belongs_to` creates an association
- `references` used in command line creates a foreign key to the articles database table; column name is `article_id` and type is integer (or in other words it creates a migration that will do that)
- now you need to run migration: `bin/rails db:migrate`

### 6.2 Associating Models

- articles to comments - 1:n relationship
- in `Comment` class has `belongs_to`
- we need to implement the other side of the association in `Article` with `has_many :comments`
- `belongs_to` and `has_many` declaration enable a lot of automatic behavior, such as `@article.comments`

### 6.3 Adding a Route for Comments

- edit `config/routes.rb` like this:


    resources :articles do
      resources :comments
    end
    
- if you execute `bin/rails routes`, you will see that these routes were added:


                  Prefix Verb   URI Pattern                                       Controller#Action
        article_comments GET    /articles/:article_id/comments(.:format)          comments#index
                         POST   /articles/:article_id/comments(.:format)          comments#create
     new_article_comment GET    /articles/:article_id/comments/new(.:format)      comments#new
    edit_article_comment GET    /articles/:article_id/comments/:id/edit(.:format) comments#edit
         article_comment GET    /articles/:article_id/comments/:id(.:format)      comments#show
                         PATCH  /articles/:article_id/comments/:id(.:format)      comments#update
                         PUT    /articles/:article_id/comments/:id(.:format)      comments#update
                         DELETE /articles/:article_id/comments/:id(.:format)      commens#destroy

### 6.4 Generating a Controller

- execute: `bin/rails generate controller Comments`


    create app/controller/comments_controller.rb
    invoke erb
    create   app/views/comments/
    invoke test_unit
    create   test/controllers/comments_controller_test.rb
    invoke helper
    create   app/helpers/comments_helper.rb
    invoke   test_unit
    create assets
    invoke   coffee
    create     app/assets/javascript/comments.coffee
    invoke   scss
    create     app/assets/stylesheets/comments.scss+
    
- add the following before the links into `articles/show`:


    <h2>Add a comment:</h2>
    <%= form_with(model: [@article, @article.comments.build], local: true) do |form| %>
      <p>
        <%= form.label :commenter %><br>
        <%= form.text_field :commenter %>
      </p>
      <p>
        <%= form.label :body %><br>
        <%= form.text_area :body %>
      </p>
      <p>
        <%= form.submit %>
      </p>
    <% end %>
    
- `form_with` here uses an array, which will build a new nested route, such as `article/1/comments` (and the form will do a POST, which will end up calling `comments#create`)
- we need to implement `create` in `app/controllers/comments_controller.rb`


    class CommentsController < ApplicationController
      def create
        @article = Article.find(params[:article_id])
        @comment = @article.comments.create(comment_params)
        redirect_to article_path(@article)
      end
     
      private
     
      def comment_params
        params.require(:comment).permit(:commenter, :body)
      end
    end
    
- each comment related needs to know which article it is attached to (we know this from `:article_id`)
- we take advantage of the methods automatically created for an association, `@article.comments.create` will create and save the comment, and automatically link it to the article
- finally, user is redirected back to the article


- in the `articles/show` we should now also display a list of comments already added for that article:


    <h2>Comments</h2>
     
    <% @article.comments.each do |comment| %>
      <p>
        <strong>Commenter:</strong>
        <%= comment.commenter %>
      </p>
      <p>
        <strong>Comment:</strong>
        <%= comment.body %>
      </p>
    <% end %>
    
## 7 Refactoring

### 7.1 Rendering Partial Collections

- we will extract comment into a partial `app/views/comments/_comment.html.erb`:


    <p>
      <strong>Commenter:</strong>
      <%= comment.commenter %>
    </p>
    <p>
      <strong>Comment:</strong>
      <%= comment.body %>
    </p>
    
- and now simply replace the code displaying comments in `articles/show` with:


    <h2>Comments</h2>
    <%= render @article.comments %>
    
- the above will render `comments/_comment` once for each comment in `@article.comments`
- as the `render` iterates over that collection, it assigns each comment to a local variable named the same as the partial (in our case `comment`), which is then available in the partial for display

### 7.2 Rendering a Partial Form

- we will also extract the form for creating new comments into a `comments/_form` partial


    <%= form_with(model: [@article, @article.comments.build], local: true) do |form| %>
      <p>
        <%= form.label :commenter %><br>
        <%= form.text_field :commenter %>
      </p>
      <p>
        <%= form.label :body %><br>
        <%= form.text_area :body %>
      </p>
      <p>
        <%= form.submit %>
      </p>
    <% end %>
    
- and replace the adding comment part in `articles/show` with:


    <h2>Add a comment:</h2>
    <%= render 'comments/form' %>
    
- `@article` is available to any partials rendered in the view because it was defined as an instance variable

## 8 Deleting Comments

- we need a link in the view (we will use `comment/_comments`) and a `destroy` action in `CommentsController`


    <p>
      <%= link_to 'Destroy Comment', [comment.article, comment], method: :delete, data: { confirm: 'Are you sure?' } %>
    </p>
    
<!---->

    def destroy
      @article = Article.find(params[:article_id])
      @comment = @article.comments.find(params[:id])
      @comment.destroy
      redirect_to article_path(@article)
    end
    
- the link in view will fire `article_comment DELETE /articles/:article_id/comments/:id(.:format) commens#destroy`
- the `destroy` action will find the article, the the comment in `@article.comments`, remove it from the database, and return us to the `article/show`

### 8.1 Deleting Associated Objects

- if you now try to delete an article that has any comments, you will get `ActiveRecord::InvalidForeignKey in ArticlesController#destroy` error
- we need to make sure that when an article is deleted, all associated comments are also deleted
- in `app/models/article.rb` you need to change `has_many` line to this: `has_many :comments, dependent: :destroy`

## 9 Security

### 9.1 Basic Authentication

- in `ArticlesController` we need to have a way to block access to the various actions if the person is not authenticated
- we can use `http_basic_authenticate_with` which can limit access to actions
- in `ArticlesController` we want user to be authenticated for every action exception `index` and `show`, add following to the top of the class:


    http_basic_authenticate_with name: "dhh", password: "secret", except: [:index, :show]
    
- in `CommentsController` we want to allow access to the `delete` action only to the authenticated users:


    http_basic_authenticate_with name: "dhh", password: "secret", only: :destroy
    
- other authentication methods are available
- two popular authentication addons are:
  - `Devise` rails engine - [https://github.com/plataformatec/devise](https://github.com/plataformatec/devise)
  - `Authlogic` gem - [https://github.com/binarylogic/authlogic](https://github.com/binarylogic/authlogic)
  
## 10 What's Next?

- continue with `Ruby on Rails Guides` - [http://guides.rubyonrails.org/](http://guides.rubyonrails.org/)
- `Ruby on Rails Tutorial` - [https://www.railstutorial.org/book](https://www.railstutorial.org/book)
  
