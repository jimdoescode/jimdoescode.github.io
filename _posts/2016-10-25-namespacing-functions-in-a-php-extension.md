---
layout: "post"
author: "jimdoescode"
title: "Namespacing functions in a PHP extension"
date: "2016-10-25 20:59:32"
tags: [C, PHP, PHP Extensions, PHP Namespacing]
---

Recently I was making a PHP extension for work and I was adding functionality to an extension that already existed. So much functionality
in fact, I ended up renaming the extension. This caused a bit of a problem come launch time. I couldn't just include my extension next to
the existing extension. They had methods that shared the same name. I needed a namespace.

We're all used to seeing PHP namespaces now. They are those funky things with the forward slashes.

```php
namespace \Foo\Bar;
```

But how do you do that in an extension? More over, how do you do that in an extension that only exposes functions not objects?

It turns out that PHP does provide this functionality, it's just not very well documented. What you're looking for is `ZEND_NS_FE` which
stands for zend namespace function entry. This nifty little call lets you define a namespace as it's first argument, the function as it's
second, and the third argument is the `ZEND_ARG_INFO` of your function. `ZEND_NS_FE` should be called inside of your `zend_function_entry`
array, the place where you'd normally use `PHP_FE`.

```c
const zend_function_entry my_functions[] = {
    ZEND_NS_FE("MyNamespace", my_first_function, arginfo_my_first_function)
    ZEND_NS_FE("MyNamespace", my_second_function, arginfo_my_second_function)
    PHP_FE_END
};
```

Once you make this extension, both functions will be under the `\MyNamespace` namespace. Meaning you need to call them as follows.

```php
\MyNamespace\my_first_function(...);
\MyNamespace\my_second_function(...);
```

So now they won't conflict with any other functions that might be in the global namespace.

Neat!
