---
layout: post
title:  My First CLI Gem
date:   2016-03-14 12:00:00 -0400
---

My first CLI gem is the top-travel-destinations-cli-gem, available at [RubyGems.org](https://rubygems.org/gems/top-travel-destinations-cli-gem) and with all code publicly available at [GitHub](https://github.com/agdavid/top-travel-destinations-cli-gem).  For a video walkthrough of the gem and an overview of the development process checkout the [YouTube](https://www.youtube.com/watch?v=FS9TE8V_6Gs) video and the post below. Enjoy!

<p>
  <span style="text-align:center; display: block;">
    <iframe type="text/html" width="682" height="414" src="https://www.youtube.com/embed/FS9TE8V_6Gs?version=3&amp;rel=1&amp;fs=1&amp;autohide=2&amp;showsearch=0&amp;showinfo=1&amp;iv_load_policy=1&amp;wmode=transparent" allowfullscreen="true" style="border:0;">
    </iframe>
  </span>
</p>


**Inspiration**

I love to travel.  With that in mind, I decided to make a gem that would assist anyone with wanderlust in getting started with planning their next adventure.  The general idea was to: (1) locate a reputable travel website; (2) identify general travel regions; and (3) list the specific recommended travel destinations within a region. TripAdvisor provided a great place to start with their [“Travelers’ Choice Destinations”](https://www.tripadvisor.com/TravelersChoice) page.

**Creating the Starter Gemfile with Bundler**

*Using Bundler for the Directory Tree.* With bundler already installed on my system from prior work, I initiated my CLI gem project with the easy-start terminal command of:

```
bundle gem top-travel-destinations  
# I later edited this root directory to be titled top-travel-destinations-cli-gem 
```

This provided me with a few of the essentials, such as a *.gemspec* file, and some of the nice-to-haves, such as a *README.md*, *CODE_OF_CONDUCT.md*, and *LICENSE.txt* file.  The  resulting generic directory tree looked, as follows:

```
├── bin 
│   ├── console
│   └── setup
├── lib
│   ├── top_travel_destinations
│       └── version.rb
│   └── top_travel_destinations.rb
├── spec
│   ├── spec_helper.rb
│   └── top_travel_destinations_spec.rb
├── .gitignore
├── .rspec
├── .travis.yml
├── CODE_OF_CONDUCT.md
├── Gemfile
├── LICENSE.txt
├── README.md
├── top_travel_destinations.gemspec
```

As described below, there were a few key changes to the above generic structure that I made in order to create a better structure for an “executable” file.

**Creating the Executable**
The purpose of the CLI gem project is to create a program executable from Terminal. I made the below changes to the generic bundler Gemfile structure to facilitate this goal.

*bin/top-travel-destinations File.*  I added the file *top-travel-destinations* to *bin* to become the key file that would be typed into Terminal to load the program (i.e., after the gem is installed, the user would type “top-travel-destinations”). The minimal ruby code is as follows:

```
#!/usr/bin/env ruby  # <= shebang identifies ruby language
require_relative '../config/environment'  # <= require loads the environment
TopTravelDestinations::CLI.new.call  # <=initializes program
```
*config/environment.rb Folder and File.* I added a folder *config* with the file *environment.rb* to list the relative dependencies of the gem. In so doing, the user loads the program from Terminal with the single ‘top-travel-destinations’ command as described above, which in turn loads this *environment.rb* and all listed dependencies. The use of an ‘environment’ reduces error by including only one place to make updates for dependencies.

*top_travel_destinations.rb.*  I deleted this file.  As created by bundler, it included a ‘require’ for the version.rb file and some namespacing code.  However, these proved unnecessary.

*top_travel_destinations.gemspec.* I made the following changes to remove and add specs to the gemspec file:

```
#spec.bindir = "exe"  # <= commented out spec which produced an error when building the gem
#spec.executables = spec.files.grep(%r{^exe/}){ |f| File.basename(f) } # <= commented out spec
```

```
spec.executables = ["top-travel-destinations"] # <= created spec to make clear the executable file in bin/top-travel-destinations
```

**The 'Brain' of the Gem: The CLI Class**

With the guidance of our instructor, [Avi Flombaum](https://twitter.com/aviflombaum), I took the “top-down” approach to this project and asked myself: *How will my user first be greeted or interact with the program?*

The CLI Class is the brain that runs the program.

The CLI Class was born out of an expectation that the user would, generally, have the following experience: (1) be welcomed to the program; (2) be shown a list of regions to select from; (3) be shown specific destinations of a selected region; (4) continue until the user exited. Thus, the #call method was created and provided an outline of other expected CLI Class methods:

```
def call
  welcome_screen  # <= says 'hi' to the user
  make_regions  # <= instantiate Region objects with first Nokogiri scraping
  add_destinations_to_regions  # <= add further attribute with second Nokogiri scraping
  list_regions  # <= call and list attribute of Region objects
  select_region # <= get user input to determine further action 
end
```

The #select_region method took user input and either provided the selected Region object’s top destinations attribute, if the user typed a region number, or closed the program, if the user typed ‘exit’.  Thus, the #select_region method contained two other methods nested inside, the #show_top_destinations and the #goodbye methods, for this functionality.

**The 'Heart' of the Gem: The Scraper Class**

The Scraper Class is the heart of the program that obtains the external information needed by the program.  It contains two class methods, each of which uses the Nokogiri gem to scrape data that becomes attributes for instances of the Region Class (further described below).

*Scraper.scrape_regions_array(index_url).* This class method is the first scraping that takes in the main website of the TripAdvisor Travelers’ Choice Destinations as an argument.  The method uses Nokogiri to locate the HTML element for each region.  The method iterates over each node, creating a hash that contains the name and url for the subpage of each region, then pushing such hash into an array.

The method returns an array of hashes – the ‘region-array’.  This region-array is later passed to the Region Class and provides two attributes (:name and :region_url) for each instance of the Region Class.

*Scraper.scrape_destinations_array(region_url).*  The second class method builds on the first scraper method by calling the subpage of each instance of Region (self.region_url).  The method uses Nokogiri to locate the html element for the top destinations of a region.  The method iterates over each node, pushing each destination into an empty array.

The method returns an array – the ‘destinations-array’. This destinations-array is later passed to the Region Class and provides a third attribute (:destinations) for each instance of the Region Class.

**The 'Soul' of the Gem: The Region Class**

As noted above, the CLI Class gives commands to make and list regions and the Scraper Class obtains the information for attributes of each region. **But where are these regions made?**

The Region Class is the soul of the program that brings objects to life, in the form of instances of the Region Class and its attributes.  Below is an outline of how commands and information are passed around within the gem.  As you read through, remember that the Scraper Class scrapes twice – first, scraping the main page and returning an array of hashes (:name, :region_url) for each region; second, scraping the subpage of the region and returning an array of destinations for each region.

```
├── CLI Class says #make_regions 
│   ├── Scraper Class uses #scrape_regions_array to scrape mainpage, and return an array of hashes with the name and subpage url for each region
│   └── Region Class uses #create_from_array to take in the array of hashes and initialize new instances of Region with a :name and :region_url
├── CLI Class says #add_destinations_to_regions
│   ├── Scraper Class uses #scrape_destinations_array to call self.region_url on each region, scrape subpage and return array of top destinations
│   └── Region Class uses #create_attribute_from_array to take in the array of destinations and add the array as a new :destinations attribute
```

The resulting Region Class has produced instances, each of which has three attributes – :name, :region_url, and :destinations.  During the #initialize, each instance region was pushed into an empty @@all class array which holds all the instances and can be called by the CLI Class, as needed, to list regions or list destinations.

**Publishing the Gem**
To publish the gem, I utilized the following commands in Terminal from the root directory (in my case, top-travel-destinations-cli-gem):

```
gem build top_travel_destinations.gemspec  #<= packaged the gem
gem push top-travel-destinations-cli-gem-1.0.0 #<= pushed to RubyGems.org
````

Note: Prior to executing the above commands, I opened a RubyGems account.

**Helpful Resources During Development**
- [RailsCasts #245: New Gem With Bundler](http://railscasts.com/episodes/245-new-gem-with-bundler)
- [RubyGems Guide: Make Your Own Gem](http://guides.rubygems.org/make-your-own-gem/)
- [RubyGems Guide: Publishing Your Gem](http://guides.rubygems.org/publishing/)
- [Blog: How to Write a Command Line Ruby Gem](http://robdodson.me/how-to-write-a-command-line-ruby-gem/)