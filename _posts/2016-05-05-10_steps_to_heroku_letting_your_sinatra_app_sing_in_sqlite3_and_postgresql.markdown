---
layout: post
title:  10 Steps to Heroku - Letting Sinatra Sing in SQLite3 and PostgreSQL
date:   2016-05-05 22:07:03 +0000
---

*The Situation:* You built a Sinatra application using SQLite3 as your database and it works perfectly on your local computer. You are excited and want the world to see it in all its glory! You decide to deploy. 


*The Problem:* You see this post on the Heroku DevCenter: [**You cannot use sqlite on Heroku**](https://devcenter.heroku.com/articles/sqlite3).


*The Solution:* Below are the 10 steps I went through to reconfigure my Sinatra application so that it could use SQLite3 in development locally and PostgreSQL in production on Heroku.  

Remember throughout this process: commit and commit often to save your progress (or, conversely, so you can revert!).




**10 Steps to Heroku**

1) **Add a Procfile** - In your top-level directory, add a file named "Procfile" with the following command: 

```
web: bundle exec rackup config.ru -p $PORT
```

This says that you want to run a web process with the command on the port that Heroku gives you, using the bundle environment given.  The "rackup" command to find the "config.ru" file should make sense since it is how we start Rack-based frameworks like Sinatra.

2) **Modify Gemfile and Delete Gemfile.lock**

In your Gemfile, add the PostgreSQL gem to the main list of gems:

```
gem 'pg'
```

But specify that the SQLite3 gem is only for development:

```
group :development do
  gem 'sqlite3'  
end
```

Then delete the existing "Gemfile.lock" file and re-bundle in terminal:

```
bundle install
```

*Why not include both gems in the main list?* Heroku would reject my push if the 'sqlite3' gem was in the main list. But, when working locally for development, there were no issues keeping the 'pg' gem in the main list. This was my solution.
 
3) **Create a config/database.yml File**

In the config folder, create a file called "database.yml". Add the following to configure the specifications of your database in development and production:

```
development:
  adapter: sqlite3
  database: db/development.sqlite
  host: localhost
  pool: 5
  timeout: 5000

production:
  url: <%= ENV['DATABASE_URL'] %>
  adapter: postgresql
  database: mydb
  host: localhost 
  pool: 5
  timeout: 5000
```

I learned a few things about these options. The ["pool"](http://stackoverflow.com/questions/12635152/what-is-the-use-of-the-pool-option-in-database-yml) is the number of simultaneous connections that your database can handle and I chose to be explicit, although Heroku defaults this to 5.  The ["timeout"](http://stackoverflow.com/questions/29179682/what-units-is-timeout-in-rails-database-yml) is the permitted time to establish a database connection and is measured in milliseconds (i.e., 5000 is 5 seconds).

4) **Modify config/environment.rb**

In the config folder, modify the "environment.rb" file.  Add the following to specify the location of your production database on Heroku:

```
configure :production do
   db = URI.parse(ENV['DATABASE_URL'] || 'postgres://localhost/mydb')

   ActiveRecord::Base.establish_connection(
     :adapter => db.scheme == 'postgres' ? 'postgresql' : db.scheme,
     :host     => db.host,
     :username => db.user,
     :password => db.password,
     :database => db.path[1..-1],
     :encoding => 'utf8'
     )
end
```

For my peers in [Learn Verified](https://learn.co/verified), the above should look familiar to the setup for specifying our sqlite database.




***Beginner Tip: Pause, Breath, Learn About Heroku***: 
At this point, your Sinatra application is almost configured for Heroku. I highly recommend pausing and learning about Heroku by going through their [Getting Started With Ruby](https://devcenter.heroku.com/articles/getting-started-with-ruby#introduction) learning module. This will get you to install and be familiar with the Heroku Toolbelt -  also it's fun!



5) **Make Your Heroku App**

In your top-level directory, create the Heroku app by entering the following command in terminal:


```
heroku create your-app-name
```


In my case I entered *heroku create sightseedc* and it generated the URL [https://sightseedc.herokuapp.com/](https://sightseedc.herokuapp.com/).

6) **Push Your Application Files to Heroku**

In your top-level directory, push your entire application to Heroku by entering the following command in terminal:

```
git push heroku master
```


Checkout your application live on Heroku by entering in terminal:

```
heroku open
```


There's an error!?!? This is OK. See below.

7) **Expect an Error, Get the PostgreSQL Addon**

Get the PostgreSQL addon by entering the following command in terminal and **keep track of the response provided**:

```
heroku addons:create heroku-postgresql:hobby-dev
```


This gets you the free Heroku hobby-dev database and provides the Heroku URL for your database. In my case, I received: HEROKU_POSTGRESQL_CRIMSON_URL

8) **Add .env File**

In the top-level directory, add a file called ".env".  In that file, set the environment DATABASE_URL key equal to the response Heroku gave you from the addon (see above Step 7):

```
DATABASE_URL=HEROKU_POSTGRESQL_CRIMSON_URL
```


Everytime the application hits ENV['DATABASE_URL] in "config/environment.rb", it will insert the above value and properly reach the location of your PostgreSQL database on Heroku. 

9) **Migrate and Seed Your Database, Save and Push Again to Heroku**

In the top-level directory, assuming you have migrations and seed data, run the Heroku versions of rake tasks to migrate and seed your database:

```
heroku run rake db:migrate
heroku run rake db:seed
```


Save and push the updated application up to Heroku:

```
git add .
git commit -m "my app for deployment on Heroku"
git push heroku master
```


10) **Enjoy Your Live Application!**

Checkout your live application using:

```
heroku open
```


Email the URL to everyone you know!


For those interested in reviewing or comparing code:
- Heroku app: [SightSeeDC](https://sightseedc.herokuapp.com/)
- Git repo: [Original SQLite3 version](https://github.com/agdavid/see-dc-sinatra-application)
- Git repo: [Heroku PostgreSQL version](https://github.com/agdavid/sight-see-dc-sinatra-application-heroku)
