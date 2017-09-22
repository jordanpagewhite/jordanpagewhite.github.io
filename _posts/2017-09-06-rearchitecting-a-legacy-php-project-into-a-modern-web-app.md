---
layout: post
title: Rearchitecting a legacy PHP project into a modern web app
categories: [php]
---

At work, we have (or perhaps I should say had) a horrible, vanilla PHP MVC (Model-View-Controller) application. There were many reasons I think it was horrible, but the most concerning one by far was that it was completely unsupportable. When I came along, it had already been sitting, almost entirely untouched, for 5 years. At this point, it's been mostly sitting for 7 years. In a way, it's impressive that it just sits and runs.

I have, and other developers have, tried to implement new features requested by internal clients, but every single time it seems to have some unintended (usually fairly silent) consequence in a completely unrelated component of the application. I am sure many of you have had similar experiences with applications like this one. This application was developed on a contract and was supposed to be a very short-term fix until we could find a good long-term solution. It has been about 7 years now. There are tons of reasons this happen (budget), and it's hard to blame anyone when things go wrong, etc. etc., but I had a border-line obsessive need to turn this application into at least a minimally acceptable web app in 2017.

## Goals

The list below isn't as exhausting as it seems. If it's too overwhelming for you, just skip to the next section when I talk about how we solved, or are working to solve, these issues.

1. Migrate from PHP 5.6 to PHP 7.0 or 7.1.
    1. Blocker: Rewrite the deprecated `mysql_` database extension
2. (PSR-4) Get every class into its own file and implement class autoloading.
3. (PSR-2) Implement some coding standards and make this application easier for other developers to support.
4. Replace PHPTemplate with Twig (yay!)
5. Replace Bootstrap with PostCSS and Gulp and remove FontAwesome and serve icons from the filesystem.
    * This will improve performance by drastically reducing the size of our assets and the call to the FontAwesome CDN, and it will also make meeting branding standards much easier/faster going forward.
6. Modularize our Javascript into seperate files based upon functionality.
7. Uglify and minify our assets with Gulp. Load the JS in the footer.
8. Generally make it easier for future developers to contribute to this codebase (particularly taking care of default settings and the Salesforce connection).

## How to turn the ship around!

### 1. Migrate from PHP 5.6 to PHP 7.0

Perhaps most importantly, there is a significant improvement in performance. Obviously, you could even update to PHP 7.1 if you'd like, but I'm going to focus on 7.0 since that was my scenario, and I think it is still very relevant for legacy 'custom', meaning not a community supported framework/CMS, PHP applications.

##### Rewrite the deprecated database extension

This particular application was using the `mysql_` database extension. That prevented me from updating to PHP 7.0. I realized that I had to rewrite it, and I ended up rewriting it using the `mysqli_` extension. There were a few reasons I chose this instead of PDO, but the biggest reason was that I didn't think I'd actually end up implementing this in production ever. I was mostly rewriting the codebase for fun, in my personal time, and I thought that rewriting from `mysql_` to `mysqli_` would be significantly faster and be easier to implement without introducing bugs.

There weren't really any 'gotcha' moments that I encountered while rewriting my database class, so I won't bother to mention the implementation. The `mysqli_` extension was very similar to the `mysql_` extension.

### 2. Implement PSR-4

This is application was created with a single `models.php` file and a `controllers.php` file. All of the model classes were in the `models.php` file, and all of the controller classes were in the `controllers.php` file. The `models.php` file was almost 2 thousand lines long. I knew immediately that I wanted to make it PSR-4 compatible. PSR-4 is a spec for autoloading classes from file paths. For my fellow Drupalers, if you've spent any time writing Drupal 8 code, you are likely already familiar with PSR-4, or you are at least already benefiting from it.

##### One class per file

To implement PSR-4, I started by taking every class in `models.php` and `controllers.php`, and I put them into their own file. Model classes went into `src/Models` and controller classes into `src/Controllers`. You can choose a better, or more descriptive, directory structure if you'd prefer of course. Then, obviously you have to delete the existing `models.php` and `controllers.php` files in the root of the project. And, boom, you've blown up the entire application. Nothing works. You are horrible at this. GAME OVER.

So, lets fix this horrible mess we created.

##### Tell Composer to use PSR-4

Create a `composer.json` file if you don't aleady have one in the root of your project. You will need to declare an `autoload` key for composer. Mine looks like this:

```json
"autoload": {
	"psr-4": {
		"": "src/"
	}
}
```

If you run `composer install` or `composer update` now, all of your PSR-4 references will be combined into a single array in `vendor/composer/autoload_psr4.php` (https://getcomposer.org/doc/04-schema.md#psr-4).

##### Aliasing/Importing

Lastly, you'll have to declare a namespace at the top of each of your model and controller files. If you maybe didn't take C++ or Java in college and aren't otherwise familiar with object oriented programming, there are tons of resources to get your started with OO in PHP (http://php.net/manual/en/language.namespaces.importing.php).

In my case, I used `namespace Model;` as my namespace for my Model classes and `namespace Controller;` for the Controller classes. You'll want to match your directory structure with your namespace for PSR-4 autoloading.

This resulted in `use` statements, in my app, like `use Model\AppBase;`

##### It's ALIVE!

After so hard work, blood, sweat and tears, I set up all of the aliasing/importing correctly. In this particular application, it was pretty tedious because there were a lot of imports to discover that, obviously, weren't there previously, and there was no documentation, so there was a lot of good ol' fashioned code reading and debugging.

### 3. Implement PSR-2

PSR-2 spec lays out some basic coding standards for PHP applications, like indendation, where to put `namespace` and `use` statements, how to define class properties and methods, and many other things.

One tool that I use all the time and that will help you format your code according to PSR-2 standards is `squizlabs/PHP_CodeSniffer` on Github (https://github.com/squizlabs/PHP_CodeSniffer). I'm also a VIM user, so I use syntastic (a syntax checker that, among many features, will help you keep your code formatted to PSR-2). https://akrabat.com/checking-your-code-for-psr-2/

### 4. Replace PHPTemplate with Twig

Twig is awesome. Introducing Twig in Drupal 8 was one of the main things that convinced me to take a deep dive into the front-end in Drupal. It feels like a normal front-end experience now (with a few Drupalism), as opposed to an entirely Drupal-specific theming/front-end process.

So, I obviously had to remove PHPTemplate from the templating system of this app in favor of Twig. To do this, download Twig with `composer require`.

After you have Twig downloaded using composer, it is really simple to configure it. You just need to add a few lines of PHP to set up the `Twig_Environment`. Here is an example of what I have in my `init.php`:

```php
$loader = new Twig_Loader_Filesystem('./views');
$twig = new Twig_Environment($loader, array(
    'debug' => TRUE,
    'cache' => './views/cache',
));
$twig->addExtension(new Twig_Extension_Debug());
```

Then, you'll have to replace do a little rewrite of whichever method(s) you have in your controller to render partials or templates. In my instance, I'm just returning `$twig->render($filename, $params)`.

Lastly, you will have to rewrite your templates in Twig. I won't go into details there since there are so many great resources online to learn Twig.

### 5. Replace Bootstrap with PostCSS and Gulp and remove FontAwesome

So, lets start with the obvious one: remove FontAwesome. It was being called from a CDN and is totally unnecessarily slowing down the app for a few social buttons. This was just an easy fix: put the icons in the app and serve them from there and remove the `<script>` referencing the FontAwesome CDN.

And now for the more complicated one. Replace Bootstrap with PostCSS and Gulp. I am not a huge fan of Bootstrap, or front-end frameworks in general. I understand their utility, but I haven't benefited from them in my particular work thus far. Instead, I vastly prefer a PostCSS set up that can be customized to the websites/application. It also should end up with much, much less CSS bloat i.e. better performance.

Again, there are tons of great tutorials to show you how to set up Gulp and PostCSS, so I won't discuss the implementation.

### 6. Modularize Javascript 

This was mostly a decision made to make working on the JS of the application easier i.e. developer experience I suppose, but it also just made sense based upon the functionality of certain 'components' within the single, long JS file.

There isn't a ton to mention with this step other than it improves the readibility of your Javascript quite a lot. It will also make it easier for our team, eventually, to remove jQuery from the application and rewrite the JS using modern ES6.

### 7. Uglify and Minify with Gulp. Load JS in the footer

First, the JS was being loaded in the head for no reason. I moved it to the footer for the performance gains.

Next, I set up Gulp so that I could uglify and minify my assets (and do a bunch of other cool stuff I won't mention).

### 8. Improve developer experience (DX)

Lastly, and this has nothing to do with performance, there were some 'tension points' in the app, particularly for onboarding developers that I wanted to resolve. The biggest one was the integration to Salesforce. In the past, there wasn't really a formal sense of environments within the app, and the app would still POST to our production instance of Salesforce even on local environments. This was unacceptable for so many reasons.

Instead, I configured environments in the settings/config file of the app so that the app would POST to a sandbox (shared amongst my team, but we could individualize it down the road) that was more or less identical to the prod instance fo Salesforce. In this way, an incoming developer will be able to just boot up the app for the first time on a vagrant box, and they won't accidentally POST Leads or any Records directly to our production instance of Salesforce.

## Performance benchmarks

Compare the two here.

## Room for improvement

As web developers, we are never done! That's the way it feels to me at least. Following that sentiment, there's still some work left to be done on the application. We can take a few seconds to pat ourselves on the back, but then it's right back to the battlefield. Here are some things I would like to tackle going forward:

1. More caching (Varnish would be awesome)
    * This app is actually very performant, but there is always room for improvement.
2. (PSR-11) Container interface
3. Implement the Symfony router in place of our custom router
4. Get rid of our jQuery dependency and get Webpack and Babel set up so we can write some modern JS.
5. 100% comment+doxygen coverage. This needs to happen. During the process of commenting what I've completed thus far (a majority), I've learned so much more about the application (obviously).
6. Many methods need to be smaller/do less things, particularly in models.

These are preliminary thoughts, and I'm sure there are many other ways to improve this app (time permitting).

## Special thanks

I want to thank my coworker, supervisor and mentor, who I will be leaving soon. Without his patience and guidance, I wouldn't be half the developer, teammate and mentor I am today. He is the greatest mentor I could have asked for in my first role as a web developer, and I really appreciate the opportunity to work with and learn alongside you.

> "I do not teach anyone I only provide the environment in which they can learn"

\- Albert Einstein (There are many variations of this quote.)

## Sources

1. [PHP FIG](http://www.php-fig.org/)
2. [Twig](https://twig.symfony.com/)

