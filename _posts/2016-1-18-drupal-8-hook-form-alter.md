---
layout: post
title: Drupal 8, hook_form_alter()
---

I am developing a Drupal 8 site right to manage scholarships and scholarship applications at the university. I wanted to add a bunch of custom fields to User and group them with the `field_group` module, but `field_group` generated an error that caused White Screen of Death (WSOD) up on adding a field group. I reported the error to the module maintainer, but he seemed to denounce the possibility that it generated an error in his response to me.

Since I am on a deadline, I decided to find a solution that didn't use `field_group`.


I'll paste my code below. I tried to add verbose comments to explain my thought process. I know that this is really only a solution to this specific problem, but I hope that this can help anyone who experiences the same issue with `field_group` that I experienced. I am sure that `field_group` will release a version to address this issue at some point and this post will become somewhat irrelevant. If anyone has an idea for a better solution to this problem, I'd be excited to hear your solution and talk about it.

```php
/**
 * Implements hook_form_alter() to add classes to the search form.
 */
function fpud8_form_user_register_form_alter(&$form, \Drupal\Core\Form\FormStateInterface $form_state, $form_id) {
  $address_types = ['permanent', 'contact'];

  /*
   * Iterates through $address_types[]
   * 1st iteration - 'permanent'
   * 2nd iteration - 'contact'
   */
  foreach ($address_types as $address_type) {
    // Create a fieldset field for the address type
    $form[$address_type.'_address'] = array(
      '#type' => 'fieldset',
      '#title' => t(ucfirst($address_type)." Address"),
    );

    foreach ($form as $key => $value) {
      /*
       * Checks each $value in $form_id for containment of the string 'field_'.$address_type.'_'
       * e.g. Does 'field_permanent_city' contain 'field_permanent_'?
       */
      if (strpos( $key, 'field_'.$address_type.'_' ) !== false) {
        /*
         * Creates [$value] as a subarray to the fieldset, $form[$address_type.'_address'],
         * that was created at the beginning of this foreach loop. This is conceptually 
         * equivalent to moving the field, $form[$value], into the $form[$address_type.'_address'] fieldset.
         */
        $form[$address_type.'_address'][$key] = $form[$key];
        $form[$key] = '';
      }
    }
  }
}
```

As you can see in the code, I need two field groups: `permanent_address` and `contact_address`, and I am finding fields to add to each field group by prepending the fields with the `$address_type`.

I am hoping that this somewhat inelegant, temporary solution will be exactly that, a good, reliable, but temporary solution. The `field_group` module is still in active development, just like almost all Drupal 8 contrib modules. I'm going to have to find some time to debug the issue myself and see if I can contribute to `field_group` myself by resolvign this issue. 
