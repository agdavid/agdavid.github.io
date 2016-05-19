---
layout: post
title:  My First Rails Application - AllergyFree Meals
date:   2016-05-18 20:49:05 -0400
---

My first Rails application is AllergyFree Meals (AFM), a web application that helps you find delicious and healthy recipes contributed by a community of allergy-conscious users.  All code is publicly available at [GitHub](https://github.com/agdavid/allergy-free-meals-rails-application) and a video walkthrough can be found on [YouTube](https://youtu.be/8eQj9awg6sI). Below is an overview of the key development processes and challenges. Enjoy!

**Inspiration for AllergyFree Meals**
Cooking is one of the daily experiences I really enjoy - it is family-centric, relaxing, but also builds up with excitment to the point of eating!  However, with a sibling managing celiac disease and a nephew severely allergic to five of the "big eight" allergens, I am also trying to make sure that dining experiences as a family are always inclusive and allergy conscious.  AFM aims to help us eat smart, eat delicious, but also eat allergy-free.

**Key Database Relationships**
AFM began as a simple application, but as the features I wanted to create grew, it blossomed into a database with nine (9) tables, including four (4) join tables.  The key join-relationships are outlined below:

1. *Items - Ingredients - Recipes*
This is the heart of the application, managing the feature that most users might look for - the ability to create a recipe. A Recipe and Item are connected through the join-table of Ingredients, and the join-table contains the extra attribute of "quantity".  As explained further in "Highlights of Coding Challenges" (below), this relationship led to a deeply-nested form and required a custom writer in the Recipe model to handle Ingredients and Items when instantiating a new Recipe.  The model relationships were as follows:

```
class Recipe < ActiveRecord::Base
  has_many :ingredients
  has_many :items, through: :ingredients
  accepts_nested_attributes_for :ingredients, allow_destroy: true
  accepts_nested_attributes:for :items
end
```

```
class Ingredient < ActiveRecord::Base
  belongs_to :recipe
  belongs_to :item 
  accepts_nested_attributes_for :item
end
```

```
class Item < ActiveRecord::Base
  has_many :ingredients
  has_many :recipes, through: :ingredients
end
```

3. *Users - FavoriteRecipes - Recipes*
This relationship creates a higher-fidelity experience by allowing a user to save recipes to a collection, creating a virtual "recipe-box" to track their favorite meals. The ActiveRecord relationships use terminology that evokes the relationship between a user and recipe. For example, to get a list of a user's favorite recipes call `@user.favorites` and to get a list of users that added a recipe to their favorites call `@recipe.favorited_by`.  The model relationships were as follows:

```
class User < ActiveRecord::Base
has_many :recipes
  has_many :favorite_recipes
  has_many :favorites, through: :favorite_recipes, source: :recipe
end
```

```
class FavoriteRecipe < ActiveRecord::Base
  belongs_to :recipe 
  belongs_to :user
end
```

```
class Recipe < ActiveRecord::Base
belongs_to :user
  has_many :favorite_recipes
  has_many :favorited_by, through: :favorite_recipes, source: :user 
end
```

3. *Allergens - RecipeAllergens - Recipes*
This relationship powers the key search feature which allows a user to filter recipes by allergen - allowing you to declare which allergens to avoid in a recipe. The model relationships were as follows:

```
class Allergen < ActiveRecord::Base
  has_many :recipe_allergens
  has_many :recipes, through: :recipe_allergens
end
```

```
class RecipeAllergen < ActiveRecord::Base
  belongs_to :recipe 
  belongs_to :allergen 
end
```

```
class Recipe < ActiveRecord::Base
  has_many :recipe_allergens
  has_many :allergens, through: :recipe_allergens
end
```

**Highlights of Coding Challenges**

1.*Overlaying Omniauth on Top of Devise and Adding Custom Attributes to Devise User*
The [Devise](https://github.com/plataformatec/devise) gem provides incredible user authentication (i.e., sign-up, login, logout) out-of-the-box, but requires some coding acrobatics when overlaying [Omniauth](https://github.com/mkdynamic/omniauth-facebook) gem authentication and adding custom attributes to a the Devise User table.  Providing Omniauth authentication means addressing additional issues of avoiding: (a) the required Devise password when authenticating using Omniauth; and (b) the Devise password confirmation when updating a User profile. On the Omniauth issue, the [RailsCast](https://www.youtube.com/watch?v=X6tKAUOMzCs) beautifully addresses these Omniauth issues.  On the custom attributes issue, the code snippets below show how I "sanitized" custom attributes by adding them to the Devise User strong parameters:

```
class ApplicationController < ActionController::Base
before_filter :configure_permitted_parameters, if: :devise_controller?
protected
def configure_permitted_parameters
devise_parameter_sanitizer.permit(:sign_up) { |u| u.permit(:email, :password, :password_confirmation, :remember_me, :name, :provider, :uid) }
devise_parameter_sanitizer.permit(:account_update) { |u| u.permit(:email, :password, :password_confirmation, :current_password, :name, { allergen_ids: [] }, :provider, :uid, :image, :motto, :admin) }
end
end
```

2.*Finding or Creating a User Instance with Omniauth*
In using Omniauth, I wanted the alternative Facebook-authentication to be *in addition* to the Devise authentication, in case a user had previously signed-up with the default Devise process.  This required a class method in the User model that could correctly locate an existing user instance by a previously existing attribute (in this case, email) and add the Omniauth attributes (provider and uid), rather than creating a new user instance.  The class method was as follows:

```
class User < ActiveRecord::Base
  def self.find_or_create_from_omniauth(auth_hash)
    where(email: auth_hash[:info][:email]).first_or_create do |user|
      #set the remaining attributes
      user.name = auth_hash[:info][:name] 
      user.provider = auth_hash[:provider] 
      user.uid = auth_hash[:uid] 
    end
  end
end
```

3.*Search Recipes by Allergen*
One of the key features of AFM is the ability to search all recipes and filter by allergens.  This was a fun use of the array "set intersection" method by which you return a new array of overlapping elements.  The approach I took was to: (a) iterate over all recipes and find the "set intersection" of recipe-allergens with the array of filtered-allergens; (b) compare the "set intersection" to the filtered-allergens; and (c) if the "set intersection" matched the filtered-allergens then push the recipe into a new collection of "matched recipes".  The class method I coded is below:

```
class Recipe < ActiveRecord::Base
  def self.match_allergens(search_allergen_ids)
    search_allergen_ids = search_allergen_ids.collect { |id| id.to_i }
    matched_recipes = []
    self.all.each do |recipe|
      intersect_ids = (recipe.allergen_ids & search_allergen_ids)  
      matched_recipes << recipe if search_allergen_ids.sort == intersect_ids.sort
    end
    matched_recipes
  end
end
```

4.*Custom Writer for Deeply Nested Form (Two-levels of Nesting)*
As noted above in "Key Database Relationships", the heart of the application is the many-to-many relationship between Recipe and Items through the join-table Ingredients.  The "new recipe form" encapsulates this relationship in a nested form with two-levels deep of nesting: (1) the ingredient, associated with a recipe; and (2) the item, associated with the ingredient.  I was able to make good use of the [Cocoon](https://github.com/nathanvda/cocoon) gem to create a dynamic nested form using jQuery.  However, the challenge was properly parsing the params hash to properly create the new or update the existing recipe in the Recipe model and then: (a) create or update the ingredient; and (c) find or create the item (i.e., look for an existing "Salt" item, if it already exists, rather than making a duplicate).  The custom writer method for the `ingredients-attributes=` I coded is below:

```
class Recipe < ActiveRecord::Base
  def ingredients_attributes=(params)
    params.each do |i, ingredient|
      # Avoid duplicating existing ingredient, if recipe is being updated
      if ingredient[:id]
        @ingredient = Ingredient.find(ingredient[:id])
      else
        @ingredient = self.ingredients.build
      end 

      # Set other ingredient attributes of amount and recipe_id
      @ingredient.amount = ingredient[:amount]
      @ingredient.recipe_id = self.id
      
      # Avoid duplicating existing item with find_or_initialize
      @item =  Item.find_or_initialize_by(name: ingredient[:item_attributes][:name].downcase.capitalize)
      @item.save

      @ingredient.item_id = @item.id 
      @ingredient.save
    end
  end
end
```

Overall, this was a great learning experience for using Rails to build a feature-rich application for the user.  I can't wait to build on this knowledge with increased user-interaction.  ***Up next: Javascript!***

