---
layout: post
title: Drupal Features, Deployment Module, and Drush Aliases
categories: [drupal, drupalplanet, drupal7]
---

Features allows you to bundle entities, and their configurations, into a feature module that is written to code. So, what is so great about that? Can't you accomplish the same functionality by building out your views, content types, etc. through the Drupal admin UI. Yes, you can, but all of the entities and configuration will be saved in the database, as opposed to in code. Saving your entities and configuration to code is an immense benefit, if not necessary, for Drupal sites that are deployed across multiple environments, and even more if there are multiple developers working on the site.

Say you're tasked with creating a staff listing feature. You could create a feature that contains a Staff content type and a Staff Listing view. Drupal will create the feature module, in code, which lets you track everything (Git, SVN, etc.). Now, using a deployment module that I will cover later, another developer can simply pull down the repo and they will have the Staff Listing feature module, with all of the functionality and configuration. Furthermore, you can just push your changes to any of your other environments (dev, stage, prod), run updb with drush (I'll cover this later), and the Staff Listing feature module will already be enabled.

### Staff Listing feature module

*I won't bother detailing the creation of the content type or view in this post. There are tons of good, verbose tutorials on those topics.*

Now, marvel at how insanely easy it is to put all of this functionality and configuration into code with a feature module.

1. Download the features module
  * drush en features -y
2. Create a new feature
3. Select the dependencies: the Staff content type and the Staff Listing view
4. Create a name, description, package, and version
  * Name: I recommend a naming convention like Staff Listing
  * Package: I would just set this to your project shortname for all of your features if you're just getting started with features. If you are working on sites with tons of features, some people suggest breaking this categorization further. I will put some references in the footer of this article.
  * Version: 7.x-1.0
5. Open up the advanced options dropdown and click 'Generate feature'

Now you have your own Staff Listing feature module that contains all of the functionality and configuration of the Staff content type and the Staff Listing view. You can check admin/modules and you will see the module you just created.

### Deployment module

Okay, so now you're probably thinking that we should just put all of our changes into our VCS and commit it to the repository and move it to a development or testing environment. There is one last thing that we can do to save all the other developers on our team time, and help prevent potential human error.

We will create a custom deployment module and use the `hook_update_N` function to automatically enable the feature module upon deployment. So, now we will actually get into our editor and write some code. Yay!

We will start by creating these 3 files:

* `site_deployment.info`
* `site_deployment.module`
* `site_deployment.install`

##### site_deployment.info

```
  name = Staff Listing
  description = Creates a Staff content type and Staff Listing view
  core = 7.x
  version = "7.x-1.0"
```

##### site_deployment.module

```php
  <?php
    /*
     * @file
     * Module file for Staff Listing
     */
  ?>
```

##### site_deployment.install

```php
  /*
   * Enable Staff Listing feature module
   */
  function themeshortname_deployment_update_7000() {
    module_enable( array(
      'staff_listing'
    ), true);
  }
```

### Using `@local`, `@dev`, `@stage`, `@prod` Drush aliases

Now you can commit all of your changes, merge your branch with master, and push your code to your development environment. We will have to run `update.php` to enable our feature module using the `hook_update_N` function in our deployment module. You could run it in the browser, but that's super lame and requires you take your hands of the keyboard. So, we will set up Drush aliases for each of our environments, which will make running `update.php` from the terminal trivial.

### Deploying your features module and deployment module

```
@dev drush updb
```

With that, your Staff Listing features module will be installed and enabled in the development environment. You can do whatever QA you have to do, and then just simply rinse and repeat for `@stage` and `@prod`.
