---
layout: post
title:  "Struggles and ActionView"
date:   2017-06-22 21:24:31 +0000
---


Getting back into a groove can be tough. This past week I did not have time to spend coding or studying. This break was at the same time as when I felt the material was really getting difficult. I am still not very confident in my understanding of nested associations.

To combat this fuzziness, I intend to thoroughly review some of these past areas which I have not had to do previously. Also, tempering expectations is important as well in the learning process. How fast one progresses in learning can change based on the material; it can change day to day.

My most recent area of struggle was working with ActionView helpers for forms. I encountered a problem as I worked on creating a form for a namespaced model.

```
namespace :admin do
  resources :preferences
end
```

This preference model exists in the admin namespace. Being used to the form_for ActionView tag I tried the following in the view.

`<%= form_for @preference do |f| %>`

This resulted in a NoMethod error as it tried to access the preference_path. Since the preference model is namespaced in admin, the proper path is admin_preference_path. To correct this, I explicitly gave the proper namespaced path as seen below.

```
<%= form_for @preference, url: admin_preference_path(@preference) do |f| %>
```
