#### Jortec Informática
# Workshop Ruby on Rails

This guide assumes that you already have a Ruby on Rails environment on your machine and that you **already know the basics of Ruby on Rails**. We will also assume that you are using **Rails 4.0.2** and at least **Ruby 1.9.3**.

## Get to know the tools

Here is the list of preferred software for this workshop:

* **Text editor** — Sublime Text ([http://www.sublimetext.com/2](http://www.sublimetext.com/2))

* **Terminal** — Where you start the Rails server and run commands. You must get used to terminal (or you can use iTerm instead).

* **Web browser** — You application will be visible on one of these. Preferably Chrome.

## Creating the application

Today we will be creating a simple web application for you to manage your daily tasks and your TODO list. This application will let you organize the tasks by category and it's able to support different users, having each one its own set of tasks.

With the help of this web application, several tricks, tips and advices for developing in Ruby on Rails will be presented, like:

* **Schema constraints**
* **Database Indexes**
* **Some gems to ease your developing**
* **User authentication (normal and with Facebook)**
* **Image uploading**
* **Pagination**
* **Query otimization (Eager loading and cache)**
* **etc..**

We're going to create a new Rails application. You can call it whatever you want. I'll just call it `life_manager`.

First open your terminal type the `rails new` command to create your new project and change current directory:

```
$ rails new life_manager
$ cd life_manager
```

### Scaffolding our application

Professional Ruby on Rails developers [usually don't use scaffold](http://stackoverflow.com/questions/6735468/why-do-ruby-on-rails-professionals-not-use-scaffolding) for numerous reasons, but it will be faster for us to rapidly implement this exercise and focus on the interesting parts. We will also use different Rails generators ahead.

So, without further ado, let's create the scaffolding for the users:

```
$ rails g scaffold User name image
```

Notice how Rails created **a lot** of files and directories. That's a reason why you usually don't want to use scaffold — most of the times you don't need them.

Ok, now create also the scaffolding for categories and tasks:

```
$ rails g scaffold Category name
$ rails g scaffold Task name description user_id:integer category_id:integer status:integer
```

Note that for `user_id` and `category_id` we had to specefy the type. We could also use `name:string` but if you don't give the type Rails will assume it as `string`.

You can now edit your `config/routes.rb` and add a root route:

```ruby
root 'tasks#index'
```

We now have a running app, where we can **create**, **update**, **delete** and **edit** tasks (CRUD).

## Edit migrations and run them

[One of the most common mistakes](http://www.mikeperham.com/2012/05/05/five-common-rails-mistakes/) when developing a Rails application is the use of migrations with no schema constraints.

We should now add constraints and defaults on migrations so our `create_tasks` migration should look like this. [We should also add indexes](http://rails-bestpractices.com/posts/21-always-add-db-index) for foreign keys, columns that need to be sorted, lookup fields and columns that are used in a `GROUP BY`.

```
class CreateTasks < ActiveRecord::Migration
  def change
    create_table :tasks do |t|
      t.string :name, null: false
      t.string :description
      t.integer :user_id, null: false
      t.integer :category_id, null: false
      t.integer :status, null: false, default: 1

      t.timestamps
    end

    add_index :tasks, :user_id
    add_index :tasks, :category_id
  end
end
```

After adding all constraints you think are relevant run `rake db:migrate` so we have our first database schema ready. For the scope of this workshop we'll use the default sqlite3 database.

## Adding development gems

As you probably know most of the times we solve common problems in Ruby on Rails with Gems. Let's open our `Gemfile` and add a bunch of gems that we'll need.

#### rails_best_practices <sub>[https://github.com/railsbp/rails_best_practices](https://github.com/railsbp/rails_best_practices)</sub>

rails_best_practices is a code metric tool to check the quality of rails code. It will help you to correct that code smell or even to learn the right way of doing things.

#### pry <sub>[https://github.com/rweng/pry-rails](https://github.com/rweng/pry-rails)</sub>

Pry is an awesome replacement for your old IRB (and also your Rails console) that adds a lot of cool features. You should [check their homepage](http://pryrepl.org/) to get a feeling on what you can do with it!

#### Better Errors <sub>[https://github.com/charliesome/better_errors](https://github.com/charliesome/better_errors)</sub>

This one is self explanatory — better Rails error page.

Add these lines to Gemfile and run `bundle install`:

```ruby
group :development do
  gem 'rails_best_practices'
  gem 'pry-rails'
  gem 'better_errors'
end
```

## User authentication

Above we created the users table so we can have the model and CRUD for users. We still need to do all the authentication layer over them and for that we will use [devise](https://github.com/plataformatec/devise).

Devise is a flexible authentication solution for Rails and it's very easy to configure. Just add this to your Gemfile:

```ruby
gem 'devise'
```

After running `bundle install` we have to run some generators installed by devise. Some gems install additional generators to your Rails environment. If you run `rails generate` you can see a list of available generators, and you can see these ones:

```
Devise:
  devise
  devise:install
  devise:views
```

Run these commands to install devise configuration files and add the required fields to the user model:

```
$ rails g devise:install
$ rails g devise User
```

After these commands you get:

* migration file to add fields to user;
* user authentication routes (via the `devise_for :users` line);
* configuration files (`config/initializers/devise.rb` and `config/locales/devise.en.yml`).

Sometimes we will need to change how the authentication forms and layouts look and so we also have to generate those views on our project:

```
$ rails g devise:views
```

You can check these files and configure things as you like but the defaults are Ok for now.

If we want to have super secret tasks on our application we should protect the tasks controller with newly configured authentication! Just add a `before_filter` on the tasks controller, like this:

```ruby
class TasksController < ApplicationController
  before_filter :authenticate_user!
end
```

When you try to access `/tasks` now you have to create a new user and sign in.

## Adding a decent layout to our application

By now you should be tired of the default looks of your application. Also you may have noticed that we are missing _logout_ that could be present on a navigation bar.

For the purpose of this workshop we're using [Bootstrap](http://getbootstrap.com/) for front-end framework:

```ruby
gem 'rails-bootstrap'
```

And we will use a very handy gem to generate a simple html layout for us:

```ruby
group :development do
  gem 'rails_layout'
end
```

To generate the layout run:

```
$ rails generate layout:install bootstrap3 --force
```

We can now edit our navigation links (edit `app/views/layouts/_navigation_links`) and add the _log out_, _login_ and _sign up_ links (with a bonus of showing you name where you are logged in):

```
<% if user_signed_in? %>
  <li><% content_tag 'a', current_user.name %></li>
  <li><%= link_to "Log out", destroy_user_session_path, method: :delete %></li>
<% else %>
  <li><%= link_to "Log in", new_user_session_path %></li>
  <li><%= link_to 'Sign up', new_user_registration_path %></li>
<% end %>
```

## Facebook authentication

Let's now add Facebook login to our application. First things first: you need to create an application on Facebook. Go to [https://developers.facebook.com](https://developers.facebook.com) and create a new application.

Then go to _settings_ and click on _add platform_ chose *Website*. You should now have a configuration windows like the one below:

![](http://cl.ly/image/2g3y0q051L3v/workshop_app_facebook.png)

We are now ready to configure the omniauth gem so we can login with Facebook using devise. Follow this tutorial and you should be fine: [https://github.com/plataformatec/devise/wiki/OmniAuth%3a-Overview](https://github.com/plataformatec/devise/wiki/OmniAuth%3a-Overview)

Don't forget to add a link to sign with facebook on your `app/views/devise/sessions/new.html.erb`:

```
<%= link_to "Sign in with Facebook", user_omniauth_authorize_path(:facebook) %>
```

## Pagination and image uploads

#### Pagination

You don't want to do `Task.all`, ever! It's Ok when you have a finite (and small) set of records but you don't know how many tasks the user will add.

For pagination we'll use _[will_paginate](https://github.com/mislav/will_paginate)_. Add it to the Gemfile:

```ruby
gem 'will_paginate'
```

Now create a lot of tasks (10 will do ;)

After having a decent number of them add pagination to your model. Change `app/controllers/tasks_controller.rb` on `index` method:

```ruby
def index
  @tasks = Task.paginate(page: params[:page], per_page: 5)
end
```
If you refresh `/tasks` you can see only five tasks per page, but to change between pages you must add the line below to you `app/views/tasks/index.html.erb`:

```ruby
<%= will_paginate @tasks %>
```

#### Image uploading

But a task, to be clearer, should have an image attached. To accomplish this we will use the _[carrierwave](https://github.com/carrierwaveuploader/carrierwave)_ gem.

First, we need to create a column for storing the image file, this will have the type string. Create a migration using `rails g migration AddImageToTasks image:string` and run `rake db:migrate`.

Next, we have to install the carrierwave gem, so add to `gem 'carrierwave'` to the gemfile and run `bundle install`. Carrierwave uses an `Uploader` class to deal with files, so we need to create one for this image. We called it ImageUploader, but you can name it what you want, run `rails generate uploader Image` to create it. This will generate a file `app/uploaders/image_uploader.rb`.

Now, you just need to add `mount_uploader :image, ImageUploader` to the Task model and image uploading is ready to go!

Don't forget to add a new field in the form for uploading the image, like:

```html
<div class="field">
  <%= f.label :image %><br>
  <%= f.file_field :image %>
</div>
```

and also a element for the image in the show view, like:

```ruby
<%= image_tag(@task.image) %>
```

## Eager loading

If you change the default `index.html.erb` to show the category name instead of the ID if you look at the rails server log you will notice that for each row you get a database request. You get **N +1** queries to retrieve all the information for the tasks.

This affects your performance! You always want to minimize the number of database requests! You can achieve this by a technique called **eager loading** of associations. When you get the tasks you also get, on the same query, the categories:

```ruby
@tasks = Task.includes(:category).paginate(page: params[:page], per_page: 5)
```

Now you have **2** queries instead of **N + 1** which is way more efficient.

## Environment variables

It's always a good idea to not store API keys, secrets, etc. in your code repository. One common solution is to store sensitive data in environment variables and not hardcoded in the source files, especially on production servers.

Although if you have multiple projects on your development machine you start to have dozens or hundreds of these variables which could be not so pleasant. For every problem there's a gem, so:

```ruby
gem 'dotenv-rails'
```

Dotenv let's you have all those variables in a file, inside the project (`.env`) which you don't add to your code repository and then access then as you would access your system environment variables (in Rails you can access environment variables with `ENV['NAME_OF_THE_VARIABLE']`).

In this example we are using Facebook authentication and we hardcoded the credentials in devises configuration file.

```ruby
config.omniauth :facebook, '286680404818637', '7b921f59e955a1872b71235ffb356da2'
```

Open that file and change it to use environment variables:

```ruby
config.omniauth :facebook, ENV['FACEBOOK_APP_ID'], ENV['FACEBOOK_SECRET']
```

Create the `.env` file in the root of your project and just add:

```
FACEBOOK_APP_ID='286680404818637'
FACEBOOK_SECRET='7b921f59e955a1872b71235ffb356da2'
```

Restart your server and you're done! Remeber to **never add the .env file to your repository**.

## The TODO list

Since we don't have much time there are a lot of topics that we couldn't cover and we would like to, like:

* **testing** your application;
* caching and optimization;
* deployment of a Rails application;
* background jobs;
* using Redis with Rails.


#### Other useful gems

* **_[Rspec-rails](https://github.com/rspec/rspec-rails)_** - great gem for creating and running tests. You really should be making tests if you're not already.

* **_[Factory girl](https://github.com/thoughtbot/factory_girl)_** - a fixtures replacement with a straightforward definition syntax, support for multiple build strategies and support for multiple factories for the same class (user, admin_user, and so on), including factory inheritance.

* **_[CanCan](https://github.com/ryanb/cancan)_** - useful and simple gem for managing permissions and authorizations within your web app.

* **_[Paperclip](https://github.com/thoughtbot/paperclip)_** - another good gem for file uploading and sometimes a good alternative for carrierwave.

* **_[Act as paroind](https://github.com/goncalossilva/acts_as_paranoid)_** - a simple gem for implementing soft delete (when you want to keep your deleted records hidden but still in the database)

* **_[Active admin](https://github.com/gregbell/active_admin)_** - a gem for generating a quick (and powerfull) backoffice.

* **_[FriendlyId](https://github.com/norman/friendly_id)_** - a gem that lets your transform your ugly links with tons of identifiers in pretty links, tailored for humans.

#### Helpful links

* **_[Ruby weekly](http://rubyweekly.com/)_** - a great newsletter about Ruby and Ruby on Rails.

* **_[Rails casts](http://railscasts.com/)_** - greate audio and video casts spanning almost all main aspects and use cases in Ruby on Rails development.

* **_[Heroku](https://www.heroku.com/)_** - a plataform that lets you put your web application online for free within minutes.
