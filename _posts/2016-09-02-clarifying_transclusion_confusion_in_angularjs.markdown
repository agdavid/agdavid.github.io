---
layout: post
title:  Clarifying Transclusion Confusion in AngularJS
date:   2016-09-02 20:49:05 -0400
---

Modules! Controllers! Directives! Oh my! The **TL;DR**: Transclusion takes the HTML content inside an element named as a directive, then inserts that HTML content into the directive's template where you designate another element named '<ng-transclude>'.  Below, I walk through my example.

The world of AngularJS has been exciting, albeit I am just scratching the surface. The ability to work with a framework on top of the JavaScript language that I already enjoy is wonderful.

One of the topics that left my scratching my head was transclusion.  Not only is the word odd, but even the explanation in the [AngularJS documentation](https://docs.angularjs.org/api/ng/directive/ngTransclude) leaves you hanging:

>Directive that marks the insertion point for the transcluded DOM of the nearest parent directive that uses transclusion.

Using 'transclusion' in the definition of 'transclusion'?!?!  I always raise my eyebrows at circular explanations that use the word they are defining!

With that, I set off to understand the basics of transclusion on my own by: (1) creating a simple index.html file; (2) creating simple directives that transclude; and (3) comparing the rendered index.html to understand how transclusion works.

**Step 1: Initial Index.html File**
I created a basic index.html file with a '<body>'' as follows:

```
<body ng-app="app">  

  <outer-directive title="OuterDirective Template">
    This will be transcluded into the template of the outerDirective directive.
    <inner-directive title="InnerDirective Template">
      This will be transcluded into the template of the innerDirective directive.
    </inner-directive> 
  </outer-directive>
  
  <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.5.7/angular.min.js"></script>
  <script src="js/app.js"></script>
  <script src="js/directives/outerDirective.js"></script>
  <script src="js/directives/innerDirective.js"></script>

</body>
```

Aside from loading the root module 'app' and the javascript files, you can see that this domain uses two directives based on the named elements. The element '<outer-directive>'' suggests an OuterDirective and '<inner-directive>'' suggests an InnerDirective.  My goal is to transclude the text (i.e., "This will be transcluded into...") inside each of the directives.  

**Step 2: Creating Directives Using Transclusion**
I could now get started on my OuterDirective:

```
angular
  .module('app')
  .directive('outerDirective', OuterDirective)

function OuterDirective() {
  return {
    restrict: 'E',
    scope: {
      title: '@'
    },
    transclude: true,
    template: [
      '<h1>{{ title }}</h1>',
      '<div ng-transclude></div>'
    ].join('')
  };
};
```

Let's walk through a few parts of the directive object returned by the InnerDirective function.  

To practice my isolated scopes, I set a scope object and used a one-way binding ('@') to set the 'title' property to the string value in the directive element (for a nice overview of scope and bindings in custom directives, check out this third-party [post](http://www.infragistics.com/community/blogs/dhananjay_kumar/archive/2015/06/11/understanding-scopes-in-angularjs-custom-directives.aspx)).

Most notably, I included 'transclude: true'.  This alerts us that we will insert the HTML contents of the '<outer-directive>' element at a certain location.

That insertion point is one that you can decide! In the template, I created a '<div>' element with the attribute 'ng-transclude' to direct Angular to insert the HTML contents of the '<outer-directive>' element at this location.

I made a similar directive for InnerDirective, also using an isolated scope, transclusion, and template with ng-transclude:

```
angular
  .module('app')
  .directive('innerDirective', InnerDirective)
function InnerDirective() {
  console.log("From the InnerDirective");
  return {
    restrict: 'E',
    scope: {
      title: '@'
    },
    transclude: true,
    template: [
      '<h2>{{ title }}</h2>',
      '<div ng-transclude></div>'
    ].join('')
  };
};
```    

**Step 3: Analyze Rendered Index.html to Understand Transclusion**
Opening up the browser, we analyze the resulting HTML markup and confirm how transclusion works in Angular.

Let's take this in parts. 

If transclusion worked, we would expect a sort of "sum": OuterDirective template + '<outer-directive>' element HTML contents at the 'ng-transclude'.

We expected the following template...

```
<h1>{{ title }}</h1>
<div ng-transclude></div>
```

plus, the below initial HTML inside the '<outer-directive>' element inserted at ng-transclude...

```
    This will be transcluded into the template of the outerDirective directive.
    <inner-directive title="InnerDirective Template">
      This will be transcluded into the template of the innerDirective directive.
    </inner-directive> 
```

resulting in...


```
    <outer-directive title="OuterDirective Template" class="ng-isolate-scope">

      <h1 class="ng-binding">OuterDirective Template</h1>
      
      <div ng-transclude="">
    
        <span class="ng-scope">
        This will be transcluded into the template of the outerDirective directive.
        </span>
    
        <inner-directive title="InnerDirective Template" class="ng-scope ng-isolate-scope">
          <h2 class="ng-binding">InnerDirective Template</h2>
          <div ng-transclude="">
            <span class="ng-scope">
              This will be transcluded into the template of the innerDirective directive.
            </span>
          </div>
        </inner-directive> 
    
      </div>
    
    </outer-directive>

```

Nice! Our expectations of transclusion worked.  I went through an analogous confirmation process for my InnerDirective directive.

Working through this simple transclusion domain was really helpful.  I would encourage all AngularJS beginners to take the time to walk through a similar example. If my sample code will help, take a look on [GitHub](https://github.com/agdavid/example-angular-transclusion).

If you have any questions, feel free to get in touch. Happy coding!
