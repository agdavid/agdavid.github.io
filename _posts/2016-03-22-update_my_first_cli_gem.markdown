---
layout: post
title:  Update - My First CLI Gem
date:   2016-03-22 12:00:00 -0400
---

<p>
  <span style="text-align:center; display: block;">
    <iframe type="text/html" width="682" height="414" src="https://www.youtube.com/embed/ZzCnz5vTyFk?version=3&amp;rel=1&amp;fs=1&amp;autohide=2&amp;showsearch=0&amp;showinfo=1&amp;iv_load_policy=1&amp;wmode=transparent" allowfullscreen="true" style="border:0;">
    </iframe>
  </span>
</p>

Since my last [post](http://agdavid.github.io/2016/03/14/my_first_cli_gem/) describing my CLI gem, the top-travel-destinations-cli-gem, I released an updated version with enhanced functionality.  Please feel free to watch the video walkthrough above or read below for more information! The gem is available on [RubyGems](https://rubygems.org/search?utf8=%E2%9C%93&query=top+travel+destinations) and code on [GitHub](https://github.com/agdavid/top-travel-destinations-cli-gem).

**Enhanced Information for the User**

The latest version (v 1.3.0) now has three levels of information for the user: (1) list of regions; (2) list of top destinations in each region; and (3) a description of each destination.  The prior version only had the first two levels of information.

**Increased Complexity With Collaborating Objects**

To achieve the third level of information (description of each destination), I added another class to the domain – Class Destination.

The prior version contained two classes – Class Scraper and Class Region.  The Scraper class contained the methods utilizing the Nokogiri scraping tool.  The Region class created instances with three attributes (:name, :region_url, and :destinations).  The :destinations attribute of a Region instance contained an array of strings; thus, this attribute was basically a list attached to each Region instance, but these destinations were not objects.

In order to give the user an enhanced experience with more information, I wanted to add the descriptive paragraph for each destination.  I created the Class Destination with three attributes (:name, :region, :description) to achieve this.

**Region “has-many” Destinations and Destination “belongs-to” a Region**

To collaborate, the two classes had to know their relationship. The code snippet below from the Class Region shows how an instance of Destination “belongs-to” an instance of Region, as well as how, each instance of Region “has-many” Destinations:
 
```
def add_destinations(destinations_array) #(1)
  destinations_array.each do |destination_hash| #(2)
  destination = TopTravelDestinations::Destination.new(destination_hash) #(3) 
    self.destinations << destination #(4)
    destination.region = self #(5)
  end
end
```

Let’s breakdown this code.

(1) An instance of Region calls the #add_destinations method, which takes an array of hashes (destinations_array) as an argument.  This array of hashes was created by scraping the Region instance webpage to get the name and destinations of each Region instance using Nokogiri. (2) The method iterates over each hash corresponding to a destination. (3) The hash is sent to Class Destination and the #initialize method is called. (4) The instance of Destination is pushed into the :destinations attribute of the instance Region creating the *“has-many”* relationship. (5) The setter method #region= for the instance of Destination is set to the instance of Region creating the *“belongs-to”* relationship.

I hope you can enjoy the improved gem. Happy travels!