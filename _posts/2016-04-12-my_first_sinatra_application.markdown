---
layout: post
title:  My First Sinatra Application
date:   2016-04-12 12:00:00 -0400
---

My first Sinatra application is SightSeeDC, a web application that helps you create a customized sightseeing plan for Washington, D.C. and also view the sightseeing plans and reviews of other users.  All code is publicly available at [GitHub](https://github.com/agdavid/see-dc-sinatra-application) and a live version is deployed on [Heroku](http://sightseedc-sinatra.herokuapp.com/).  Below is a video walkthrough of SightSeeDC and an overview of the development process. Enjoy!

<p>
  <span style="text-align:center; display: block;">
    <iframe type="text/html" width="682" height="414" src="https://www.youtube.com/embed/sXplBn7gSdI?version=3&amp;rel=1&amp;fs=1&amp;autohide=2&amp;showsearch=0&amp;showinfo=1&amp;iv_load_policy=1&amp;wmode=transparent" allowfullscreen="true" style="border:0;">  
    </iframe>
  </span>
</p>

**Inspiration for SightSeeDC**

Continuing the travel-related theme, I wanted to help people make custom sightseeing plans, kind of like a Burger King-esque “have it your way” of touring.  Crowdsourcing has turned into a powerful tool, as exhibited by any ratings-based application such as Yelp, TripAdvisor or AirBnB, and I wanted to incorporate reviews into this application.  The general idea was to: (1) create a CRUD application using Sinatra, (2) permit users to create custom lists of sights, (3) submit reviews for sights, and (4), as the crowdsourcing component, look at plans and reviews of other users.

**Creating the Basic Directory Structure**

Before creating the custom SightSeeDC application, I created the following generic directory structure to make sure I could run a basic Sinatra application (step-by-step instructions follow):

```
├── config
│   └── environment.rb
├── app 
│   ├── controllers
│       └── application_controller.rb
│   ├── models
│   └── views
├── db  
├── config.ru 
├── Rakefile 
├── Gemfile
```

*Main Directory* – I created a directory with the name “see-dc-sinatra-application” to hold all files.

*Gemfile* – From within the main directory, I entered `bundle init` in Terminal to create a Gemfile.

*config.ru* – In the main directory, I created the config.ru which runs the Sinatra application, with the following code:

```
require './config/environment' #requires environment

if ActiveRecord::Migrator.needs_migration? #reminder to run migrations
  raise 'Migrations are pending. Run 'rake db:migrate'
end

use Rack::MethodOverride #for later use of patch/delete routes
runApplicationController #mounting main controller
```

*config/environment.rb* – In the main directory, I created the “config” folder and “environment.rb” file to load dependencies, such as gems, databases, and the MVC files, with the following code:

```
ENV['SINATRA_ENV'] ||="development"

require 'bundler/setup' #require bundler
Bundler.require(:default, ENV['SINATRA_ENV']) #load gems, using bundler

ActiveRecord::Base.establish_connection(
  :adapter => "sqlite3",
  :database => "db/#{ENV['SINATRA_ENV']}.sqlite"
) #identify the adapter and create connection for database

require_all 'app' #require all MVC files in the app folder
```

*app* – In the main directory, I created the “app” folder to hold the MVC folders of “models”, “views”, and “controllers”.  I added only the folder “controllers” and the file “application_controller.rb” with the following code to run the test application:

```
class ApplicationController < Sinatra::Base
  get '/'do
    "Hello World"
  end
end
```

*Rakefile* – In the main directory, I created a “Rakefile” withe the following code to provide access to certain rake tasks:

```
ENV['SINATRA_ENV'] ||="development"

require_relative './config/environment'
require 'sinatra/activerecord/rake' #provide access to activerecord rake tasks
```

*gems* – I included 11 gems in my Gemfile: sinatra, activerecord, sinatra-activerecord, rake, require_all, sqlite3, thin, shotgun, pry, bcrypt, tux. From within the main directory, I entered `bundle install` to install the gems.

After completing the above generic structure, I entered `shotgun` from within the main directory. I navigated to the local server (localhost:9393) and confirmed that the main index route (‘/’) was rendering the text “Hello World.”

**Models and Object Relational Mapping**

*Models* – The application with users, sights, and reviews created by users that pertain to sights lead to four models:

- class User – has many reviews, has many sights through a join table

- class Review – belongs to a user, belongs to a sight

- class Sight – has many reviews, has many users through a join table

- class UserSight (the join table) – belongs to a user, belongs to a sight

*Migrations* – The table for each model consisted of the following attributes

- users – username, email, password_digest

- reviews – content, user_id, sight_id

- sights – name, description

- user_sights – user_id, sight_id

**Controllers and RESTful Routing**

I separated concerns between four controllers, as follows:

*ApplicationController* – set configurations (e.g., views and public folder), index routes, helper methods

*UsersController* – inherits from ApplicationController, then takes care of sign-up, log-in and log-out, and creating, editing, and showing user sights

*SightsController* – inherits from ApplicationController, then takes care of creating and showing sights

*ReviewsController* – inherits from ApplicationController, then takes care of creating, editing, deleting and showing reviews

Remember, that after creating controllers you must “mount” them in config.ru, as follows:

```
use UsersController #use all other controllers
use ReviewsController 
use SightsController 
runApplicationController #mount main controller
```

*RESTful Routing.* This is a RESTful application and, where applicable, the controllers have the conventional routes, as follows:

- UsersController – GET request to render a user show page, GET and POST create requests to render and process a new form for a user’s sightseeing list, GET and PATCH edit requests to render and process an update form for a user’s sightseeing list

- ReviewsController – GET request to render show pages for all and individual reviews, GET and POST create requests for new reviews, GET and PATCH edit requests for existing reviews, DELETE request for existing reviews

- SightsController – GET request to render show pages for all and individual sights, GET and POST create requests for new sights

**Views and Bootstrap Framework**

During this project, I was most excited to do two things within Views: (1) use layouts to implement the DRY principle; and (2) use Bootstrap for a stylish and responsive website.

*Layouts* – I wanted pages to have repeated styling and the “layout” feature of Sinatra helped me in that regard. However, I could not rely on a single “layout.erb” file because I wanted two different layouts depending upon whether a user was logged-in.  I created a “views/layout” folder and two different layouts – external.erb and internal.erb.

* *external.rb* – the layout used at the homepage, “Log In” and “Sign Up” pages before getting to the main website
* *internal.rb* – the layout used at all other web pages once logged-in

An example of calling the desired layout follows:

```
get '/' do
  if logged_in
    #other code
    erb :"/index", :layout => :"layout/internal"
  else
    erb :"/homepage", :layout => :"layout/external"
  end
end
```

*Bootstrap* – I wanted to create a responsive website using the Bootstrap framework. To implement this, I created a “public” folder in the main directory and placed the Bootstrap HTML and CSS styling in that folder.

Overall, this was a really incredible process that brought together the front-end (HTML, CSS and Bootstrap) and back-end (Ruby) using the Sinatra framework. ***I’m excited to keep on moving because right around the corner is Rails!***