---
layout: post
title: Drupal 8 Remove textarea formats and About text formats
categories: [drupal, drupal8]
---

I'm working on a project now in which a particular role is restricted to a single text format. I was surprised to find that (even when a role is restricted to a single text format) the 'About text formats' link still shows.

![before](https://www.drupal.org/files/issues/Create%20Article%20%7C%20Site-Install%202015-01-24%2001-22-12.jpg)

Image credit: drupal.org

I thought that this was confusing and unnecessary, so I decided to remove it. Here is a very simple way that I removed the link and the unnecessary box around the link.

Create a custom module and open the .module file.

### .module file

```php
function _allowed_formats_remove_textarea_help($form_element, FormStateInterface $form_state) {

  if (isset($form_element[0]['format'])) {
    // All this stuff is needed to hide the help text.
    unset($form_element[0]['format']['guidelines']);
    unset($form_element[0]['format']['help']);
    unset($form_element[0]['format']['#type']);
    unset($form_element[0]['format']['#theme_wrappers']);
    $form_element[0]['format']['format']['#access'] = FALSE;
  }

  return $form_element;
}
```

You'll have to add an `#after_build` entry for `_allowed_formats_remove_textarea_help()` to each textarea fvalue in whichever form(s) you want to hide the 'About text formats'.

```php
$form['field_your_textarea_machine_name']['widget']['#after_build'][] = '_allowed_formats_remove_textarea_help';
```

### Permissions

I have a few roles in my application, so I thought it would be better to control this functionality with a permission as opposed to explicitly stating the roles in the .module file.

### .module file

```php
$user = Drupal\user\Entity\User::load(\Drupal::currentUser()->id());

if (!$user->hasPermission('hide text formats and about text formats')) {
```

### .permissions.yml

```yaml
'hide text formats and about text formats':
  title: 'Hide text formats, "About text formats" and the surrounding box'
```

### Sources
[Issue d8 on d.o.](https://www.drupal.org/node/2413335)  
[Thread on d.o. for solution in d6 and d7](https://www.drupal.org/node/215653)  
[Simplify module](https://www.drupal.org/project/simplify)
