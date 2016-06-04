---
layout: post
title:  Ruby 101 - Creating An Executable
date:   2016-03-28 12:00:00 -0400
---

This post is focused on structuring a simple “ready-to-run” Ruby program executable from the command line (Terminal in Mac OSX) and dedicated to all the beginning Rubyists. I hope this is helpful for beginners getting a project off the ground!

For the TL;DR check out my executable-ruby-template on [GitHub](https://github.com/agdavid/executable-ruby-template)! But, for beginning programmers, the explanation below may be helpful.

When first learning how to program, it is easy enough to run your Ruby file from within the root directory that holds your files.  For example, if my file was called *‘name.rb’*, I would type the following into Terminal from within the root directory:

```
ruby name.rb
```

But, at some point, you may want your program to be “ready-to-run” and executable by a normal user from their ‘bin’. Here is how I learned to structure my directory:

```
├── bin 
│   ├── console
│   └── name #<=rename this to match your program
├── config
│   ├── environment.rb
├── lib
│   └── name.rb # <=rename this to match your program
├── Gemfile
├── Gemfile.lock
```

Let’s break this down, going in order of how this structure logically came about:

**lib/name.rb** – This is the file you are probably most familiar with as a beginning Rubyist; the bread-and-butter code, such as “puts ‘Hello World'”. Here, it is located in the ‘lib’ folder.

**bin/name** – This is the entry point for the user from Terminal into your program – the goal is to be able to type *‘./bin/name’* to execute your program.  This provides separation so that the user does not interact directly with your Ruby code. This file has two lines of code:

```
#!/usr/bin/env ruby
require_relative '../config/environment'
```

The first line tells the interpreter that this is Ruby language. The second line requires the ‘config/environment’, which loads our file dependencies, such as ‘lib/name’.

TROUBLESHOOTING: *I get an error when trying to run the file with ‘./bin/name’.* The likely suspect is that the file is not executable. Go into your ‘bin’ directory and type in Terminal ‘ls -lah’ – this will return a list of all files and permissions in human readable format. If your ‘bin/name’ file does not have an ‘x’, type *‘chmod +x name’* to make it executable.

**config/environment** – This is your central point for loading your Ruby file dependencies – i.e., by requiring this one file, you get all of your code! This file has three lines of code:

```
require 'bundler'
Bundler.require
require_all './lib'
```

The first line requires the [Bundler](http://bundler.io/) gem, the amazing gem for loading gems in our Gemfile.  The second line actually requires the gems in the Gemfile. The third line requires the files in the ‘lib’ directory, using the ‘require-all’ command of the [Require-All](https://github.com/jarmo/require_all) gem. Basically, this ‘environment’ gets everything loaded into memory so your program can run!

**Gemfile** – This is where you list any required gems for the bundler gem. Something for beginning Rubyists to remember is that the Gemfile holds a list of gems, but does not load them – it is the job of bundler to load these gems.

**bin/console** – This is your “sandbox” – enter it by typing in Terminal *‘./bin/console’*. For troubleshooting on making it executable, see above.  The console is not part of your program, but is a GREAT means to interact and experiment with your objects by loading an interactive Ruby (irb) session. The file has four single lines of code and a #reload! method:

```
#!usr/bin/env ruby
require 'irb'

def reload!
  load_all './lib'
end

require_relative '../config/environment'
IRB.start
```

The first line of code tells the interpreter this is Ruby language. The second line requires irb. Skipping the method for now, we then require ‘config/environment’ to load file dependencies and start the irb session.

**The #reload! method** – By typing *‘reload!’* into your irb session, this method reloads your file dependencies including any edits/changes saved. Did you just debug an error by adding a missing “end” or “comma”? Did you add a new attribute to your class? This method uses the ‘load_all’ command of the require_all gem and saves you time by avoiding the hassle of having to restart a new irb session. Just reload and keep experimenting.