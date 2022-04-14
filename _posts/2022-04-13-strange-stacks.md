---
layout: post
title: "Evolving Stacks"
subtitle: "When should legacy web sites start from scratch?"
date: 2020-01-28 23:45:13 -0400
background: '/img/posts/03.jpg'
---

In my work I often encounter legacy websites that were built well over a decade ago. 
As I review the source, I see the developers made efforts 
each year to modernize the codebase.

Unfortunately, generations of changes are grafted onto the same structure. This sometimes leads to a very 
Frankenstien-like code beast. 
I'm guilty of building the same. For years I built full-stack web applications using Django, and as 
trends evolved, the libraries I used to support the HTML, CSS, and JS I wrote changed over time.

Django uses **server-side** templates, which are rendered by the server. Though Django templating syntax is more like Handlebars
or Mustache.js, django followed in a long tradition of **server-side** templating. Remember CGI and Perl scripts? These, along with other granddaddies like PHP or ASPs or JSPs dynamically munged HTML/CSS with data on the server side, and sent to web browsers as fully rendered HTML files. 

In the 2010's client-side rendering grew in popularity and eventually took over. Javascript in the early 2000's
was not standardized as each browser had its own implementation. Glue libraries like JQuery 
provided an abstraction layer that made JS code more portable.

AngularJS made a major splash in October, 2010, and server-side developers looked for ways to integrate it into 
their existing frameworks. Some split template-rendering into client frameworks that communicated with their
servers via Restful APIs. Others build server-side toolchains that compiled AngularJS into static assets to served
by their backends.

Angular 2 was a big departure from Angular 1, giving some teams pause to consider other frameworks like React and Vue.
Some teams started migrating from Angular 1, never to completely abandon it.


## Groovy on Grails

I've been looking at code that was written in Groovy on Grails, the Ruby on Rails inspired JAVA-based web framework.

Over time a number of technologies were employed, including:

* Java 8 (the language)
* Tomcat 8 (the webserver)
* MySQL (the database)
* GSP (Groovy Server Pages)
* Gradle (build tool)
* bower (javascript package manager)
* ElasticSearch ( indexing / search )
* Docker ( container with everything preconfigured for use )
* Angular 1 ( web framework -> compiled to server static assets )
* Coffeescript ( little language that compiles to javascript )
* Asset Pipeline ( framework for minifying static assets, compiling coffeescript to js )

The issue I was brought on to troubleshoot was related to Angular compile errors that had cropped up after working to upgrade to Grails 5.

This error in question was in the form "XXX is not using explicit annotation and cannot be invoked in strict mode".

The solution was to declare all dependency injections in string array. Any code like this... 

```javascript
angular.module('my.forms.actions', ['my.ui.actions' ]).config(
  function (etcProvider ) {
  "ngInject"
    ...
});
```

...must be updated like this:

```javascript
angular.module('my.forms.actions', ['my.ui.actions' ]).config(
  ['etcProvider', function (etcProvider ) {
  "ngInject"
    ...
}]);
```

The same error appeared in several coffeescript files. It was trickier 
to find the equivelant fix using coffeescript so I opted to 
[decafinate](https://decaffeinate-project.org/) the script into standard es6 JS.

Dozens if not hundreds of similar fixes were needed, but why? Why now? Apparently in previous iterations of the code, the Angular source was not in [STRICT](https://www.youtube.com/watch?v=QkC1hjXx0dU)  mode. 

It turns out that a GSP file had been introduced a few weeks prior containing a **ng-strict-di** tag. Removing the tag removed the strict mode compilation errors.


```html
 <div id="demoCurator" ng-app="demoCurator" ng-strict-di>
```

In a case like this, if sticking with Angular 1, it might be beneficial to embrace strict mode to help future developers avoid syntax mistakes. But the project owner wisely decided not to invest the time, as they're planning to move all of the Angular and Coffeescript code to React soon.

## Lessons learned

The initial build errors told us exactly where to look for the problem but not how to fix it. We could have spent days refactoring code to rid the source of this error. 

Niche or boutique programming languages don't always have a long shelf life. While researching solutions, all proposed solutions and examples were written in Javascript, which meant additional time was needed to translate the solutions to coffeescript. Coffeescript seemed a viable language a few years ago, and some developers found it less cumbersome than standards-based JS. It was good for them. But was it good for those who follow long after the alternative language has faded? How much quality documentation will be written in an aging syntax?

The build chain and package management was a mix of JAVA-related technologies and JS-related technologies, which is necessary if your stack is built on both languages. But it would be useful to do a 50,000 foot analysis and ask, "Is there an less verbose, less complex, easier-to-maintain way to provide this same functionality in 2022?" 
