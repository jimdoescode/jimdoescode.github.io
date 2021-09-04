---
layout: "post"
author: "jimdoescode"
title: "Unknown named parameter $routes in RequestResponseArgs.php"
date: "2021-08-03 13:50:14"
tags: [php, php8, slim]
---

After upgrading to PHP 8 on a server where I was running a [Slimframework V3](https://www.slimframework.com/docs/v3/) application I started receiving this mysterious error.

The error was "Unknown named parameter $routes" occuring on line 39 of `Slim\Handlers\Strategies\RequestResponseArgs.php`. Here's what that line looks like:
```php
return call_user_func_array($callable, $routeArguments);
```

I'm using `RequestResponseArgs` to have named parameters in my route functions. It just makes things so much easier to follow so taking it out wasn't really an option. It would also require a major rewrite.

Since I didn't write that class and it worked just fine in PHP 7.4 my programmer spidey sense said this was something that had changed in PHP 8. The problem was it wasn't immediately obvious what had occurred. In that line there is no variable named "`$routes`" and the `call_user_func_array` method didn't look like it was being used improperly. So what's the deal?

After some searching around I came across [this article](https://chrislloyd.co/fixing-laravel-php-8-error-unknown-named-parameter-error/) which was describing a similar problem in Laravel. It mentioned that there was a change in PHP 8 to the way `call_user_func_array` and similar methods work. [Here is the description of that change](https://wiki.php.net/rfc/named_params#call_user_func_and_friends). 

Basically it boils down to how `call_user_func_array` operates when given an associative array. Prior to PHP 8 array keys would be ignored when passing in an associative array. Now, with named parameters, array keys matter on that associative array. So in our example we have an array with a key named "routes" but the function we're calling doesn't take a parameter named "routes" so PHP 8 throws an error. 

This is a quite simple fix, strip the keys from the associative array:
```php
return call_user_func_array($callable, array_values($routeArguments));
```   

Slim framework 3 lets you use whatever route strategy class you'd like so I made a copy of `Slim\Handlers\Strategies\RequestResponseArgs.php` adding in the `array_values` call and now PHP 8 is happy 👍 
