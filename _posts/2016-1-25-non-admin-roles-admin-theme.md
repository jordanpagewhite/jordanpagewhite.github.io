---
layout: post
title: Non-admin roles adding/editing content using admin theme
categories: [drupal, drupal7, drupal8, drupalplanet]
---

I ran into a recent surprise while working with some content editors on an academic catalog website. I sat down with them to walk through some issues they experienced with speciic content editing tasks. To my surprise, when they went into the Edit view, ex. /node/1/edit, the university theme was still being used instead of the admin theme (I think I was using Seven or Admininal). I was pretty sure that I clicked the checkbox to use the admin theme on content editing pages on the Admin > Appearance page. Let me paste a screenshot below.

![_appearance-page]({{ site.baseurl }}/images/appearance-page.png)

So, it turns out I did check that box, so how could it be possible that these content editors weren't being served the admin theme on content editing screens? There is the following option on the Permissions page:

![_permissions-page]({{ site.baseurl }}/images/permissions-page.png)

I hope that this can help someone else out there who hasn't run into this problem before. This was the first Drupal website that I developed at the university that had non-admin content editors. All of the other Drupal websites that I've developed at the university have had their content managed by my department by users with admin, or near admin, roles.

**UPDATE:** I just added an issue to d.o., [issue 2651228](https://www.drupal.org/node/2651228), to add a brief description on the Appearance page.

### Sources

I have to give some obligatory credit to my tallest coworker, Lead Developer, rival, BFF, and the oldest web developer at the university, Justin Gable, for directing me to this permission. THNX D00D!
