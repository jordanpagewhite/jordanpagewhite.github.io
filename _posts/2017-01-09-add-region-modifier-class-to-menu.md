---
layout: post
title: Add region modifier class to menu
categories: [drupal, drupal8, drupalplanet]
---

I asked this in the DrupalTwig slack channel a couple days ago:

>I’m trying to figure out how to allow for different menu styles depending upon which region a menu is placed in. Has anyone run into this issue?

>I already have my 3 menu markup/styles in PatternLab and I just need to figure out how to do the Drupal side

Here's what I came up with to resolve my issue. It is definitely far from flawless. I will try to continue to improve it, but here it is for now. Please let me know if you have a better way to add a modifier class to menus based upon the region they are placed in.

{% highlight php %}
function hook_preprocess_block(&$variables, $hook) {
  $block_id = $variables['elements']['#id'];
  $block = \Drupal\block\Entity\Block::load($block_id);

  if (strpos($block->getPluginId(), 'system_menu_block') !== FALSE) {
    $variables['content']['#attributes']['class'] = "menu--" . str_replace('_', '-', $block->getRegion());
  }
}
{% endhighlight %}

It would be much more ideal to have a region variable in menu.html.twig.

## Sources

1. [http://kristiankaa.dk/drupal8-add-class-to-menu](http://kristiankaa.dk/drupal8-add-class-to-menu)
2. [http://kristiankaa.dk/article/2015/06/27/drupal8-region-specific-menu-theme-hook-suggestion](http://kristiankaa.dk/article/2015/06/27/drupal8-region-specific-menu-theme-hook-suggestion)

