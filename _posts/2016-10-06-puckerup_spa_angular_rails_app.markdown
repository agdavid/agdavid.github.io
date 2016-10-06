---
layout: post
title:  PuckerUp - My SPA Built With AngularJS and a Rails API
date:   2016-10-06 20:49:05 -0400
---

<p>
  <span style="text-align:center; display: block;">
    <iframe type="text/html" width="682" height="414" src="https://www.youtube.com/embed/9N_rNwhFwl0?version=1&amp;rel=1&amp;fs=1&amp;autohide=2&amp;showsearch=0&amp;showinfo=1&amp;iv_load_policy=1&amp;wmode=transparent" allowfullscreen="true" style="border:0;">
    </iframe>
  </span>
</p>

Learning AngularJS was amazing, plain and simple.  It was incredible to finally see how you could plan out and engineer a single-page application (SPA) with a dynamic AngularJS frontend that communicates with a Rails API backend.  I understood how you could split up architecting between frontend and backend teams.

But let's get into the project.

**Idea:** 
PuckerUp is my latest application and it's a SPA! It brings together my excitement for AngularJS, Rails and sour beers.  The goal of PuckerUp is to provide users with the ultimate, crowd-sourced list of sour beer breweries across the USA.  Everywhere you go, PuckerUp will help you locate your next sour brew.

For the quick overview, please enjoy my video walkthrough above.  More detailed discussion of architecting the application is below.  Enjoy!


**Design:** 
In terms of design, PuckerUp is the marriage between two standalone components: (1) the Rails API backend; and (2) the AngularJS frontend.

The Rails API is a lean and mean database that provides JSON when needed.  The ORM includes the following key relationships:

```
class Brewery < ApplicationRecord
    belongs_to :state, optional: true
end

class State < ApplicationRecord
    has_many :breweries, dependent: :destroy
end
```
*Why the 'optional: true' for a Brewery?*  
Rails 5 made the 'belongs_to' association required by default. You can find this in the initializer:

```
#config/initializers/active_record_belongs_to_required_by_default.rb
Rails.application.config.active_record.belongs_to_required_by_default = true
```

This means that if the associated record has not been instantiated prior to instantiating the primary record, Rails will not persist the primary record to the database.  In my scenario, since a Brewery belongs_to a State, the State had to exist before instantiating the Brewery, otherwise ActiveRecord would rollback the database insertion.

You can make this relationship optional.  I chose to do that on my model by adding 'optional: true'.

*Dealing with Cross Site Request Forgery*
When working with Angular and Rails, you need to address the possibility of cross site request forgery (CSRF) attacks.  In general, CSRF is whan an attacker tricks the browser of a verified user into attacking a website that relies on user-authentication for certain requests.  A user that saves some verification to the browser (i.e., a cookie) could unknowingly send a malicious HTTP request toa site that trusts the user. Read more about CSRF, generally, [here](https://en.wikipedia.org/wiki/Cross-site_request_forgery).

The Rails-community-solution to protecting your API is to circularly pass an identifier (in this case called the 'CSRF token') into the cookie of each request received, then have the frontend send the same token back to Rails on each subsequent request, as verification. 

Rails is saying, "I'll tell you the password, give it back to me next round and we can do business again."  See this [Stackoverflow](http://stackoverflow.com/questions/7600347/rails-api-design-without-disabling-csrf-protection) discussion and the implementation below:

```
#in application_controller.rb
class ApplicationController << ActionController::Base
    #prevent CSRF attacks
    protect_from_forgery with: :exception
    #after each action set the verifying cookie
    after_filter :set_csrf_cookie

    #sets the verifying cookie to the key of 'XSRF-TOKEN' which Angular looks for
    def set_csrf_cookie
        cookies['XSRF-TOKEN'] = form_authenticity_token if protect_against_forgery?
    end

    #Angular sends back the cookie with the key 'X-XSRF-TOKEN'
    #but Rails is looking for a key named 'X-CSRF-TOKEN'
    protected
    #the below informs Rails that 'X-XSRF-TOKEN' is still satisfactory verification
        def verified_request?
            super || valid_authenticity_token?(session, request.headers['X-XSRF-TOKEN'])
        end
end

#in your Angular root module
  angular
      .module('app', [])
      .config(function($httpProvider) {
          //passback the cookie on $http requests in the headers
          $httpProvider.defaults.headers.common['X-CSRF-Token'] = $('meta[name=csrf-token]').attr('content');
      });

```



