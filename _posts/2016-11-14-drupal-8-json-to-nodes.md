---
layout: post
title: Drupal 8, Turn JSON into Nodes in 5 minutes
categories: [drupal, drupal8]
---

Preface: Lol, whoops. Forgot to post this for many, many months. Here it is for the sake of posterity. Hopefully it is still beneficial to someone.

Yesterday, I worked briefly on migrating a Drupal 6 site to Drupal 8. I pondered what is the preferred method of migrating content types, content, taxonomies, menus, etc. I decided just to give it a shot and this is what I came up with for migrating Nodes. I will add another post if/when I have a chance to migrate the taxonomies and menus.

For those trying to recreate this process for a Drupal 6 to 8 migration, you'll have to create a View to export your content to CSV on your Drupal 6 site and then create the content types on your new Drupal 8 sites. Get the CSV file and convert it to JSON, then follow the steps below to import the JSON as nodes. There are probably other ways to do this, but this method was insanely fast and, at least in the case of the site I was working on, it satisfied all my requirements.

I'm going to create a custom module that will have a config form with one field that will accept the URL path to the JSON file (in my case on my localhost). Then, we will handle the creation of nodes in the submission handler of that form.

### Create a module

```bash
drupal generate:module
```

### Create a config form to hold the url to import JSON from

```bash
drupal generate:form:config
```

Drupal Console will scaffold a lot of the form for you. I titled my form class BaseUrlForm. Since we are using the Node class, we will need to add `use Drupal\node\Entity\Node` at the top of our file. If you generated your form field in the Drupal Console dialogue, you will already have your `$form['baseurl']` field in `buildForm`. If not, just add the code now.

We are going to add our code to the `submitForm` function because we want Drupal to pull in this JSON content and save each entry as a node upon the submission of the form. I'm also going to create a private method to create the nodes and separate some of the logic in the submission handler.

### submitForm()

First, lets take care of our form field. We want to save it into configuration.

```php
$this->config('artimporter.baseurl')
  ->set('baseurl', $form_state->getValue('baseurl'))
  ->save();
```

Lets set `$baseurl` to the string value of 'baseurl'.

```php
$baseurl = $this->config('artimporter.baseurl')->get('baseurl');
```

Lets use a try-catch statement to handle exceptions. In the try statement, we'll get the `$response` from `$baseurl` using `\Drupal::httpClient()->get()`. To get the body of the response, so we'll set `$data` to `$response->getBody()`. If `$data` is empty, we can't do anything, so lets set a message using `drupal_set_message`. In a real application, there are better ways to handle this and you would probably want to check some other things like that status code, but we'll ignore those for now.

If we get some data, we'll call `$this->createArt($data)`. So, now we have to create that method. Since we will only use it in this context, lets make it protected.

```php
try {
  $response = \Drupal::httpClient()->get($baseurl, array('headers' => array('Accept' => 'text/plain')));
  $data = $response->getBody();

  if (empty($data)) {
    drupal_set_message('Empty response.');
  }
  else {
    $this->createArt($data);
  }
}
catch (RequestException $e) {
  watchdog_exception('artimporter', $e);
}
```

Here is our `createArt` method. First, we will use `json_decode` to decode the data that we received from the http client. I am passing the `TRUE` parameter because I wanted it to be converted to associative arrays.

Then, we'll loop through each item and create a node. You can see that my content type is 'art', and I'm setting the values of each of my fields in the last 4 entries of the array passed to `Node::create`. Lastly, don't forget to save the node!

```php
/**
 * Create nodes from JSON feed.
 */
protected function createArt(string $json) {
  $jsonout = json_decode($json, TRUE);

  foreach ($jsonout as $art) {
    $node = Node::create(array(
      'type' => 'art',
      'langcode' => 'en',
      'uid' => '1',
      'status' => 1,
      'title' => $art['Title'],
      'field_appraised_value' => $art['Appraised value'],
      'field_artist' => $art['Artist'],
      'field_quantity' => $art['Quantity'],
    ));

    $node->save();
  }
}
```

Full `src/Form/BaseUrlForm.php` code:

```php
<?php

/**
 * @file
 * Contains \Drupal\artimporter\Form\BaseUrlForm.
 */

namespace Drupal\artimporter\Form;

use Drupal\Core\Form\ConfigFormBase;
use Drupal\Core\Form\FormStateInterface;
use Drupal\Core\Config\ConfigFactoryInterface;
use Symfony\Component\DependencyInjection\ContainerInterface;
use Drupal\node\Entity\Node;

/**
 * Class BaseUrlForm.
 *
 * @package Drupal\artimporter\Form
 */
class BaseUrlForm extends ConfigFormBase {

  public function __construct(
    ConfigFactoryInterface $config_factory
  ) {
    parent::__construct($config_factory);
  }

  public static function create(ContainerInterface $container) {
    return new static(
      $container->get('config.factory')
    );
  }

  /**
   * {@inheritdoc}
   */
  protected function getEditableConfigNames() {
    return [
      'artimporter.baseurl',
    ];
  }

  /**
   * {@inheritdoc}
   */
  public function getFormId() {
    return 'baseurl_form';
  }

  /**
   * {@inheritdoc}
   */
  public function buildForm(array $form, FormStateInterface $form_state) {
    $config = $this->config('artimporter.baseurl');
    $form['baseurl'] = array(
      '#type' => 'textfield',
      '#title' => $this->t('baseurl'),
      '#description' => $this->t(''),
      '#default_value' => $config->get('baseurl'),
    );

    return parent::buildForm($form, $form_state);
  }

  /**
   * {@inheritdoc}
   */
  public function validateForm(array &$form, FormStateInterface $form_state) {
    parent::validateForm($form, $form_state);
  }

  /**
   * {@inheritdoc}
   */
  public function submitForm(array &$form, FormStateInterface $form_state) {
    parent::submitForm($form, $form_state);

    $this->config('artimporter.baseurl')
      ->set('baseurl', $form_state->getValue('baseurl'))
      ->save();

    $client = \Drupal::httpClient();
    $baseurl = $this->config('artimporter.baseurl')->get('baseurl');

    try {
      $response = \Drupal::httpClient()->get($baseurl, array('headers' => array('Accept' => 'text/plain')));
      $data = $response->getBody();

      if (empty($data)) {
        drupal_set_message('Empty response.');
      }
      else {
        $this->createArt($data);
      }
    }
    catch (RequestException $e) {
      watchdog_exception('artimporter', $e);
    }
  }

  /**
   * Create nodes from JSON feed.
   */
  protected function createArt(string $json) {
    $jsonout = json_decode($json, TRUE);

    foreach ($jsonout as $art) {
      $node = Node::create(array(
        'type' => 'art',
        'langcode' => 'en',
        'uid' => '1',
        'status' => 1,
        'title' => $art['Title'],
        'field_appraised_value' => $art['Appraised value'],
        'field_artist' => $art['Artist'],
        'field_quantity' => $art['Quantity'],
      ));

      $node->save();
    }
  }

}
```
