---
layout: post
title: Drupal 8 assets, libraries, and tracking assets
categories: [drupal, drupal8, drupalplanet]
---

There are many different ways to include assets in Drupal 8, and there were probably just as many or more ways in Drupal 7, but the recommended method in Drupal 8 is to use your theme's `theme.libraries.yml` file. I'm going to explain the method that my coworker and I are using to include css and js libraries and how we get the into version control. This is the best method, for our use case(s), that I have discovered thus far in tinkering with Drupal 8, but I would be very excited to hear anyone's opinions or suggestions in the comments.

At work, I use Grunt, Bower, Zurb Foundation 5 (6 was recently released), Modernizr, and other stuff, so I will use these libraries as examples, but the same principles apply for other libraries.

### The problem

When we download Foundation 5 and Modernizr, the assets will be placed in subdirectories of `bower_components`. Since we don't need to track the `bower_components` directory, we can copy the files that we need to either the /js or /scss directory. This is really easy with [grunt-contrib-copy](https://github.com/gruntjs/grunt-contrib-copy) and it looks like Gulp has a similar plugin, [gulp-copy](https://www.npmjs.com/package/gulp-copy).

### Gruntfile.js

Here's the code in my `Gruntfile.js` to copy over the Foundation assets and Modernizr assets:

```js
copy: {
  dist: {
    files: [
      {expand: true, cwd: 'bower_components/foundation/js/', src: ['**'], dest: 'js/'},
      {expand: true, cwd: 'bower_components/foundation/scss/', src: ['**'], dest: 'scss/'},
      {expand: true, cwd: 'bower_components/modernizr/', src: ['modernizr.js'], dest: 'js/modernizr/'},
      {expand: true, cwd: 'bower_components/modernizr/', src: ['grunt.js'], dest: 'js/modernizr/'},
      {expand: true, cwd: 'bower_components/modernizr/feature-detects/', src: ['**'], dest: 'js/modernizr/feature-detects/'}
    ]
  }
},
```

### theme.libraries.yml

Then I can declare Foundation and Modernizr as libraries in my theme's `theme.libraries.yml` file:

```yaml
foundation:
  version: VERSION
  js:
    js/foundation.min.js: {}

modernizr:
  header: true
  version: VERSION
  js:
    js/modernizr/modernizr.js: {}

global-js:
  version: VERSION
  js:
    js/app.js: {}
  dependencies:
    - fpud8/foundation
    - core/jquery
```

You can see that I used `header: true` to load Modernizr in the head. I also included Foundation's js as a dependency to my global-js because I want it to load before my custom Javascript.

### .gitignore

Now, the `.gitignore` file in your theme's root directory can include `bower_componenets/` and `node_modules/`, which is in the `.gitignore` file included with Foundation anyway.

### That's all

I really hope that this post was helpful to you. Again, I really want to hear how everyone else is handling front-end file structure and version control issues, like this one. Please let me know!
