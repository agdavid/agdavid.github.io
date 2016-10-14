---
layout: post
title:  GoogleMaps JavaScript API and Rails - Part 1 of 3
date:   2016-10-14 20:49:05 -0400
---

Numbers and words aren't always enough to sate a user's appetite.  This is where data-visualization comes in.  Let's use Rails and the GoogleMaps JavaScript API to serve up some mapped-out data for visual consumption.

This post will have three main parts: (1) saving mapping data to your Rails back-end; (2) using AJAX to retrieve that mapping data in the front-end; and (3) creating a map with the GoogleMaps JavaScript API.

Let's get started!

**Part I: Saving Mapping Data to Your Rails Back-End:**

To plot a coordinate with the GoogleMaps API you'll need both a latitude and longitude.  

At first glance, you might think: Coordinates are numbers with trailing decimals. That sounds like a good use of the decimal data type in my migration...NOPE.  Remember that the internet is just a bunch of information being shared in string format (don't tell the cats).  Information saved in decimal format will be serialized and lose the trailing decimals when received on the front-end.

The latitude and longitude should be saved as string datatype, containing however many decimals you would like to retain.  This results in the following for your coordinates:

*Migration File*
```
# db/migrate/<crazy_rails_number>_create_coordinates.rb

class CreateCoordinates < ActiveRecord::Migration[5.0]
  def change
    create_table :coordinates do |t|
      t.string :lat
      t.string :lng

      t.timestamps null: false
    end
  end
end

```

*Routes*
```
# config/routes.rb
...
resources :coordinates, only: [:index]
...
```

*Model*
```
# models/coordinate.rb
class Coordinate < ApplicationRecord
end
```

*Controller*
```
# controllers/coordinates_controller.rb
  def index
      @coordinates = Coordinate.all 
      respond_to do |format|
        format.json { render json: @coordinates }
      end
  end
```

Now, remember again that data is sent across the internet in string format.  This is where ActiveModelSerializers come in -- helping us serialize our data so that you can call on the API (coordinates#index) and recieve JSON.

Go ahead and add the proper gem to your Gemfile and rebundle:
```
# Gemfile
...
gem 'active_model_serializers'
```

Generate a serializer for our coordinates with the generator, by running in terminal:
```
rails g serializer coordinate
```

You will now have a new folder called 'app/serializers' with a file named 'coordinate_serializer.rb'. Magic! Let's take a look at this file.

```
# serializers/coordinate_serializer.rb
class CoordinateSerializer < ActiveModel::Serializer
  attributes :id
end
```
The generated file automatically includes the id of your instance.  We want to include the other attributes in our serialized data so type in the ":lat" and ":lng".  The final file should look like:

```
class CoordinateSerializer < ActiveModel::Serializer
  attributes :id, :lat, :lng
end
```

Go ahead and start up your Rails server...

```
rails s
``` 
... and navigate to localhost:3000/coordinates.json . You should get something rendered in your browser like this:

```
[
  {
    id: 1,
    lat: "33.45115165",
    lng: "-84.47746155"
  },
  {
    id: 2,
    lat: "34.04145679",
    lng: "-84.06980447"
  },
  {
    id: 3,
    lat: "34.00760354",
    lng: "-84.4051481"
  }
  ...
]
```

Voila!  At this point, you have a fully-functional Rails API.  Calls to the API endpoint 'localhost:3000/coordinates.json' will be served serialized coordinates as data.

Coming Up: Part 2 - Using AJAX to Retrieve Data in the Front-End  
