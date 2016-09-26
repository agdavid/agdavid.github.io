---
layout: post
title:  Three's Company - Working with Heroku, Paperclip and Amazon S3 to Save Images
date:   2016-09-26 13:58:33 -0400
---

If a picture is worth a thousand words, let's make sure your Heroku-deployed app can actually save and load images using our three friends: Heroku, Paperclip and Amazon S3!

*Note: This post deals with integrating Amazon S3 into an app that already uses Paperclip. Learn how to use Paperclip with your Rails models [here](https://github.com/thoughtbot/paperclip) -- the documentation is awesome!* 

Working locally is a breeze.  You use the Paperclip gem and your photos magically appear on your development application hosted on the local server.

Working on Heroku is a little different.  If you try to load an image, it works -- for about 24 hours and then disappears.  Why? Heroku doesn't want to be a free cloud storage service for you.

**For the win, you need to use [Amazon S3 (Simple Storage Service)](https://aws.amazon.com/s3/) together with your Heroku app and Paperclip gem.**

Let's do this now!

1) **Create Your Amazon S3 Account**

It's free and easy. Sign up for [Amazon Web Services here](https://aws.amazon.com/free/). 

2) **Establish Your Amazon S3 Credentials**

You need credentials for your app to access your Amazon S3 storage. 

In the upper right, navigate to the dropdown under your name > "Security Credentials".  Select "Access Keys (Access Key ID and Secret Access Key)" > "Create New Access Key". 

**Save these credentials!** You will need them for your ENV variables.
 
3) **Create a Bucket**

You need a specific place for your app to save its images.  Amazon S3 calls this a "bucket".

In the upper left, naviage to the dropdown under "Services" > "S3".  Click on the button "Create Bucket".

*Tip:* Follow these conventions for [naming buckets](https://devcenter.heroku.com/articles/s3#naming-buckets) from Heroku. 

4) **Update that Gemfile**

You need to add the Amazon Web Services gem to your Gemfile.

After trial and error, there were issues with my dependencies and I had to specify versions of the Paperclip, Postgres and Amazon Web Services gems, as follows:

```
gem 'paperclip', '4.3.6'
gem 'pg', '~> 0.18.4'
gem 'aws-sdk', '<2.0'
```

Let's rebundle -- simple enough:

```
bundle install
```

5) **Update Production Environment File**

You need to let Paperclip know that images will be stored with Amazon by setting configuration defaults.

In config/environments/production.rb, add the following:

```
# Amazon S3
  config.paperclip_defaults = {
  storage: :s3,
  s3_credentials: {
    bucket: ENV['AWS_S3_BUCKET'],
    access_key_id: ENV['AWS_ACCESS_KEY_ID'],
    secret_access_key: ENV['AWS_SECRET_ACCESS_KEY']
  }
}
```

6) **Commit, Commit, Commit to Git**

Easy and standard (and perhaps you have already committed many times before this prompt!):

```
  git add .
  git commit -m “<your message>” 
```

7) **Set Heroku ENV variables**

As always, you don't want your credentials in your git repos.  This is why we use the ENV hash and set our actual values to named keys!

Remember those ID, KEY and bucket name from before? Let's set the Heroku ENV variables now:

```
heroku config:set AWS_S3_BUCKET=your_bucket_name
heroku config:set AWS_ACCESS_KEY_ID=your_key
heroku config:set AWS_SECRET_ACCESS_KEY=your_secret
```

Confirm you set them by checking:

```
heroku config
```

Read more about this on the [Heroku Dev Center](https://devcenter.heroku.com/articles/config-vars).

8) **Push Your Application Files to Heroku**

In your top-level directory, push your entire application to Heroku by entering the following command in terminal:

```
git push heroku master
```

9) **Check Out Your App Live!**

Let's check out your live app again -- now with image-saving functionality!

```
heroku open
```

Here is the live [AllergyFreeMeals](https://allergyfreemeals.herokuapp.com/) app. Here is the [GitHub](https://github.com/agdavid/allergy-free-meals-rails-application-heroku) repo.

I welcome any questions or concerns.  Please feel free to contact me at antonio.david.us@gmail.com. Happy coding!