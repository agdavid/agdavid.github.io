---
layout: post
title:  Playing Catch - Understanding Rails FormHelpers
date:   2016-04-27 12:00:00 -0400
---

I’ve been in the land of Ruby on Rails for about one-month now. Basic and nested forms, and the use of the all-powerful FormHelpers, was the topic that I slowed down for, in order to get a deep understanding.

***Why?***  Forms are an integral part of our experience as users of web applications.  Whether it’s signing into Facebook, Gmail or another portal for online content, we interact with forms as users all the time. Therefore, it is important to understand forms as developers to make for a positive user experience.

***What’s the difficulty?*** Rails provides an incredible set of tools for working with forms and ActiveRecord models in an efficient way through FormHelpers. But the details of the underling HTML are abstracted away into “magic”.  The key is understanding how these FormHelpers work so that they can work for you.  Between the three aspects of your MVC-framework application, you need to understand exactly how information is being received and how it is being sent between the view and the controller.

So how do you master Rails FormHelpers…

**Play Catch!**

By this, I mean practice a few simple steps of: (1) anticipating the HTML that the FormHelper generates; (2) anticipating the params hash that will be sent from the View to the Controller. In so doing, you know what information the form is throwing and the controller is receiving – hence, catch!

Let’s do an example using a tag of the form_for method and a simple domain of a Book model, where @book instance is being created with a :title attribute.

** 1. What is the View throwing in params?**

The abstracted tag of “text_field”…

```
<%=f.text_field :title %>
```

…which generates the following HTML with the key information in the name attribute (book[title])…

```
<input type="text" id="book_title" name="book[title]" value="#{@book.title}" />
```

…which helps us anticipate the form of the params hash, which we need to get the actual value of the :title attribute…

```
{'book' => {'title' => ‘My Title'}}
```

** What is the Controller catching in params?**

The Controller method to create the @book instance retrieves the value in params as follows…

```
params[:book][:title]
```

…which means we need to make sure to whitelist our strong params appropriately…

```
params.require(:book).permit(:title)
```

This process of “playing catch” and anticipating the HTML and params hash has really helped me understand and work with FormHelpers.