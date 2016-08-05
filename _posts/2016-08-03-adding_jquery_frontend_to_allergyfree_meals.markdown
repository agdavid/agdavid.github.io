---
layout: post
title:  Adding a jQuery Front End and JSON API Back End
date:   2016-08-03 20:49:05 -0400
---

Since my last [post](http://agdavid.github.io/2016/05/19/my_first_rails_app_allergyfree_meals/) describing the development of AllergyFree Meals (AFM), I expanded on AFM by adding dynamic features through the use of jQuery and a JSON API for the app.  A video walkthrough of the new features is available below and all code is publicly available at [GitHub](https://github.com/agdavid/allergy-free-meals-rails-application).

<p>
  <span style="text-align:center; display: block;">
    <iframe type="text/html" width="682" height="414" src="https://www.youtube.com/embed/9ngnO9sCZGQ?version=1&amp;rel=1&amp;fs=1&amp;autohide=2&amp;showsearch=0&amp;showinfo=1&amp;iv_load_policy=1&amp;wmode=transparent" allowfullscreen="true" style="border:0;">
    </iframe>
  </span>
</p>

I've always known that JavaScript was the means to create dynamic front-ends, but this was a great chance to apply my JS skills and see them in action.  Through this process I was able to render index and show pages via jQuery without a page refresh, create JSON responses using Active Model Serializers (AMS), and translate those JSON responses into JS Model Objects which included methods on the prototype.

**Generating List of a User's Recipes on the Show Page**
From a user's profile page, I thought that generating a list of that user's recipes was a logical functionality to expect.

*Adding JS Hooks to User's Show Page* First, I created the link with a class of 'js-showUserRecipes' which, when clicked, would trigger the AJAX request:

```
<%= link_to "Show Contributed Recipes", user_recipes_path(@user), class: "btn btn-primary js-showUserRecipes", :'data-id' => @user.id %>
```
I also created the HTML element where I would drop the JSON response for the AJAX request:

```
<ol class="userRecipes">
  <!-- JSON response will go here -->
</ol>
```

*Coding JS Event Handler to Load User and Recipes* Second, I attached an event handler function, which would fire when the link was clicked.  Generally, the function would prevent the page refresh, fire an AJAX GET request to retrieve the user (including recipes) as JSON, translate the JSON response into a JS Model Object, then use a method on the JS prototype to reveal the recipes before dropping them into the DOM:

```
$(function() {
  // attach event handler function
  $(".js-showUserRecipes").on("click", function(click) {
    // render response without page refresh
    click.preventDefault();
    var userId = parseInt($(click['target']).attr("data-id"));
    var recipes_html = ""

    // translate JSON into JS Model Object
    function User(id, name, recipes) {
      this.id = id
      this.name = name
      this.recipes = recipes
      // method on the prototype
      this.display_each_recipe = function() {
        // reveal has_many recipes relationship
        $.each(this.recipes, function(i, recipe) {
          recipes_html = recipes_html.concat("<li><a href='/users/" + userId + "/recipes/" + recipe.id + "'>" + recipe.title + "</a></li>")
        });
      };
    };

    // get JSON
    $.get("/users/" + userId + ".json", function(data) {  
      var user = new User(data['id'], data['name'], data['recipes']);
      user.display_each_recipe();
      $('.userRecipes').html(recipes_html);  
    });

  });
});
``` 

*Creating JSON API Back-End* Third, I created the API which would provide the JSON response using AMS.  The above AJAX GET request was directed to the users#show action which was refactored to provide not only HTML, but also JSON using the respond_to method:

```
def show
    @user = User.find(params[:id])
    ...
    respond_to do |format|
      format.html { render :show }
      format.json { render json: @user }
    end
  end
``` 

The request auto-magically provides the user response in JSON using AMS.  In this case, my JSON user object provided the id, name, and the further-serialized recipes and comments of the user:

```
class UserSerializer < ActiveModel::Serializer
  attributes :id, :name
  has_many :recipes
  has_many :comments
end
```

This was not the only feature I added. I went through a similar process loading a comments#show page via jQuery and a JSON API back-end.  That was also incredibly fun because, before clicking on the link for comments#show, I generated the comments#index using the 'remote true' paradigm.  ***We can talk about Remote True in another post!***  
