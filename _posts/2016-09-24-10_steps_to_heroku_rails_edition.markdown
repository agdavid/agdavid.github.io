---
layout: post
title:  10 Steps to Heroku - Rails Edition
date:   2016-09-24 13:58:33 -0400
---

Hi there! Let's get your Rails application up and running on Heroku in 10 steps!

As developers, we spend a lot of time coding in our text editor.  But the way the world interacts with our code is through live applications.  What better way to showcase your work than letting it shine on Heroku. 

**10 Steps to Deploying on Heroku**

1) **Create Your Heroku App**

In your local application directory create the Heroku app by executing the following command: 

```
heroku create <optional app name>
```

Heroku will happily provide you with some random name (e.g., damp-coast-4529.herokuapp.com ... seriously) so go ahead and choose something you like instead. My command looked like this:

```
heroku create allergyfreemeals
```

Confirm that you successfully added the Heroku remote repo with the git command:

```
git remote -v
```

One of your repos should now reference Heroku!

2) **Modify Gemfile**

In your Gemfile, add the PostgreSQL gem to the production group and specify the SQLite gem for development and test -- Heroku uses Postgres and not SQLite for databases:

```
group :production do 
  gem 'pg'
end

group :development, :test do 
  gem 'sqlite3'
end
```

Lastly, there is a Heroku gem to add which is helpful for debugging.  The 12 factor gem allows you to see the full logs in standard out (STDOUT) that you are used to seeing in your local console in development. Add the following:

```
group :production do
  gem 'rails_12factor'  
end
```
 
3) **Modify config/database.yml File**

In the config folder, navigate into the database.yml file. Update the production configuration to specify the use of Postgres in production:

```
production:
  adapter: postgresql
  database: mydb
  pool: 5
  timeout: 5000
```

4) **Update that Gemfile**

Let's rebundle -- simple enough:

```
bundle install
```

5) **Precompile Assets**

Prior to building my app, I knew that I wanted to use [FontAwesome](http://fontawesome.io/icons/).  Apparently, people tend to run into issues where their icons would NOT show up in production.  The solution was to precompile assets:

```
rake assets:precompile RAILS_ENV=production
```

6) **Commit, Commit, Commit to Git**

Another easy one (and perhaps you have already committed many times before this prompt!):

```
  git add .
  git commit -m “<your message>” 
```

7) **Push Your Application Files to Heroku**

In your top-level directory, push your entire application to Heroku by entering the following command in terminal:

```
git push heroku master
```

7) **Migrate and Seed Your DB**

Assuming you have migrations and seed data, run the Heroku versions of rake tasks to migrate and seed your database:

```
heroku run rake db:migrate
heroku run rake db:seed
```

8) **Update OAuth Domains and Callbacks**

Note: This step only applies to you if you use any third-party authentication, such as OAuth for Facebook.

In development, I used my local server and whitelisted both the domain (http://localhost:3000) and callback (http://localhost:3000/users/auth/facebook/callback) on [Facebook](https://developers.facebook.com/).

With your app live on Heroku you need to update these, otherwise your "Login with Facebook" button won't work.

In my app on the Facebook developer site, navigate to Settings > Basic and update your "App Domain". Mine changed to: 

```
https://allergyfreemeals.herokuapp.com/
```

Then navigate to Products > Facebook Login and add the new callback to "Valid OAuth Redirect URIs". Mine changed to:

```
https://allergyfreemeals.herokuapp.com/users/auth/facebook/callback
```

9) **Add OAuth Keys and Secrets to ENV**

Note: Again, this step only applies to you if you use any third-party authentication, such as OAuth for Facebook.

We already know that registering your app with a provider like Facebook, Github, or Google provides you with identifiers like an 'app key' or 'app secret'.

Assuming you used key-value pairs ENV['<INSERT_KEY/SECRET_NAME>'], as any safe programmer does (!), we need to set those in our Heroku app.

I called my Facebook identifiers ENV['FACEBOOK_KEY'] and ENV['FACEBOOK_SECRET'] and set them as follows:

```
heroku config:set FACEBOOK_KEY=1234
heroku config:set FACEBOOK_SECRET=abcd
```

Confirm you set them by checking:

```
heroku config
```

Read more about this on the [Heroku Dev Center](https://devcenter.heroku.com/articles/config-vars).

10) ** Check Out Your App Live! **

The moment you've been waiting for. Let's check out your live app!

```
heroku open
```

Email the URL to everyone you know!


For those interested in reviewing or comparing code:
- Heroku live app: [AllergyFreeMeals](https://allergyfreemeals.herokuapp.com/)
- Git repo: [Original SQLite3 version](https://github.com/agdavid/allergy-free-meals-rails-application)
- Git repo: [Heroku PostgreSQL version](https://github.com/agdavid/allergy-free-meals-rails-application-heroku)
