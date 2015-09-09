---
layout: post
author: jim
title:  "Get React working with Browserify and NPM without Gulp or Grunt"
date:   2015-07-18 13:50:47
tags: [js, javascript, react, browserify, npm, uglifyjs, reactify, watchify]
---

Why are there so many build tools in the front-end world? It seems needlessly complex. Anyway I 
have decided to take a simpler approach with my latest React project.

Every time I Googled "React Browserify" the top posts would contain some kind of grunt or gulp script. 
I never understood why that was. Neither of those really bring anything to the table when it comes 
to compiling JSX, they only provide hooks to let Browserify do all the work. This made it really 
challenging to cut through the cruft and figure out what was really going on. Not to mention the huge 
number of dependencies they require.

I decided to take working with React JSX one step at a time. The first step meant using a tool that 
would convert my JSX into standard javascript. This lead me to Browserify which has a nice plugin for 
React called Reactify. Reactify will compile your React JSX into a single javascript file and 
provides a nice dependency tree using Nodes `require` function. 

To run Browserify with Reactify execute the command:

{% highlight bash %}
$ browserify -t reactify path/to/jsx/files -o public/script.js
{% endhighlight %}

Once you get this working it's generally all you need. That said there are a few extra tools and 
plugins for Browserify that come in handy. One of the nicest from a development standpoint is 
Watchify. It's a Browserify plugin that will watch whatever files you tell it and anytime there is a 
change it recompiles everything into a new script.js file. The tricky thing with it is that
it basically takes the place of the Browserify command. 

{% highlight bash %}
$ watchify -t reactify path/to/jsx/files -o public/js/script.js
{% endhighlight %}

Browserify also has an uglify plugin but I was never successful in getting that to work with Reactify.
I ended up going with the standalone Uglifyjs tool. Fortunately all the Browserify stuff writes to stdout 
so piping output in bash is straight forward. 

{% highlight bash %}
$ browserify -t reactify path/to/jsx/files | uglifyjs > public/script.min.js
{% endhighlight %}

To bring all these commands together we can use the scripts hash in NPM's package.json file. This will
let us perform any of those commands using `npm run`. First we add them to the scripts hash

{% highlight javascript %}
"scripts": {
    "build-dev": "browserify -t reactify path/to/jsx/files -o public/script.js",
    "build-prod": "browserify -t reactify path/to/jsx/files | uglifyjs > public/script.min.js",
    "watch": "watchify -t reactify path/to/jsx/files -o public/script.js",
    "clean-js": "rm public/*.js"
},
{% endhighlight %}

Now any of the following commands are available:

{% highlight bash %}
$ npm run build-dev
{% endhighlight %}
Will compile our JSX into a single script file.

{% highlight bash %}
$ npm run build-prod
{% endhighlight %}
Will compile our JSX into a single script file then uglify it.

{% highlight bash %}
$ npm run watch
{% endhighlight %}
Will compile our JSX into a single script file anytime one of our JSX files changes.

{% highlight bash %}
$ npm run clean-js
{% endhighlight %}
Will remove all js scripts from our public folder. 


Working with Browserify and NPM this way should be pretty understandable. This setup also has far fewer 
dependencies than gulp or grunt, which are both pretty dependency heavy.