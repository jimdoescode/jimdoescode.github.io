---
layout: "post"
author: "jimdoescode"
title: "Run composer install from code"
date: "2020-02-11 13:50:14"
tags: [php,composer,deployment]
---

If you've written any PHP in the last 10 years, you've more than likely used [Composer](https://getcomposer.org). It's PHP's de facto dependency manager. And sure, you could take the easy route and just run the shell command, but where's the fun in that? It's way cooler to actually "drive" Composer from your PHP code.

## Step 1: Installing Composer?

So you might think you have Composer installed for your project, but you actually need to install Composer's code as a dependency to your project. Meaning: You're going to use Composer to install Composer. 

NOTE: I'm writing all my Composer commands as though I have Composer globally installed, because I do, but if you're running it directly from your project, then run the Composer phar as you would normally.

```sh
$ composer require composer/composer
```

Once that shell command completes, you should have Composer's code as a dependency to your project. 

## Step 2: Finding Composer's home

Depending on where in your code you're actually driving Composer--in my case it's several directories down--you'll need to navigate PHP to the project root directory. If your code **is** executing from the root directory, you can skip this part.

```php
chdir(__DIR__ . '/../../../');
``` 

Great. Now PHP should be executing from your project root. The next task is to determine where Composer actually lives in the file system. 

If you've got the Composer phar sitting right in your project root, then you can skip this part. 

If, like me, you have Composer installed globally, then you need to hunt for it.

```php
$output = [];
exec('which composer 2>&1', $output, $exitCode);
if ($exitCode !== 0 || empty($output)) {
    error_log('Could not locate composer using `which composer`');
    exit 1; // Or return false or throw an exception or whatever
}
```

All we're doing here is running `which composer` via the shell. That should output the location of the Composer phar in our file system. Again, if you don't have it installed globally on your system or you're sure that it will **always** be installed in the same place on every machine your code will run on, then you can skip this part.

Now that we have the path to the Composer phar file, we need to set the constant that Composer uses to orient itself during run time. 

```php
putenv("COMPOSER_HOME={$output[0]}");
```

When we run Composer the `Composer\Factory::getHomeDir()` method needs to have this environment variable set or the run will fail. This is where you can hardcode the location if you know it and are sure it will never change.

## Step 3: Instantiate Composer

Pretty self-explanatory. We need to get an instance of Composer so we can then run the commands we want.

```php
$composer = new ComposerApp();
$composer->setAutoExit(false);
```

Note that second line, where we call `setAutoExit(false)`. It's pretty important. If we don't do that, Composer will call PHP's `exit` function and halt our script's execution after any commands are run. We don't want that to happen because we might still have stuff we want to do after we run a Composer command. So make sure you set auto exit to false.

## Step 4: Run the damn thing

Okay, we're all set. Let's make something for Composer to output to and then run a command.

```php
$stream = fopen('php://temp', 'w+');
$exitCode = $composer->run(
    new \Symfony\Component\Console\Input\ArrayInput(['command' => 'install']),
    new \Symfony\Component\Console\Output\StreamOutput($stream)
);
$result = stream_get_contents($stream, -1, 0);
fclose($stream);
```

We're opening a writable stream to the temp directory, running Composer's install command, then pulling the contents of the stream into a variable before closing the stream. Now the `$result` variable should have all the output you normally see when running Composer via the command line. Finally, you can check the value of `$exitCode` to see if Composer executed successfully. The value will be `0` if it was successful.

## Wrap up

Here's our full example. (Assuming Composer is globally installed)

```php
chdir(__DIR__ . '/../../../');

$output = [];
exec('which composer 2>&1', $output, $exitCode);
if ($exitCode !== 0 || empty($output)) {
    error_log('Could not locate composer using `which composer`');
    exit 1; // Or return false or throw an exception or whatever
}

putenv("COMPOSER_HOME={$output[0]}");

$composer = new ComposerApp();
$composer->setAutoExit(false);

$stream = fopen('php://temp', 'w+');
$exitCode = $composer->run(
    new \Symfony\Component\Console\Input\ArrayInput(['command' => 'install']),
    new \Symfony\Component\Console\Output\StreamOutput($stream)
);
$result = stream_get_contents($stream, -1, 0);
fclose($stream);
```

Now you should be able to run Composer from inside your PHP app, which is handy for things like automatic deployments.  
