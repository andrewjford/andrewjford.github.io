---
layout: post
title:  Expense Tracker Sinatra App
date:   2017-05-27 05:58:18 +0000
---


I have (for the most part) completed my first Sinatra app. It took me just over a week to put together, with much of the last few days spent tinkering with css and the design. The current functionality of the app allows users to track their spending. The app lets the user record expenses by date,category, and description. The users can then easily view monthly reports of their spending grouped by category, as well as other reports filtered by category or description.

The current version of the app is built on a sqlite database and uses bcrypt for passwords. I intend to deploy the app to heroku in the near future, so look out for a link in a subsequent post.

My greatest struggle during the process was definitely with the front-end. As alluded to above, it took me a number of days to get the design and styling to a point I found acceptable. This was due to several reasons. For one, I was rusty with my front-end skills. It had been a while since I had worked on making things pretty in css. Another reason was I did not want to make the site with Bootstrap. I wanted to have a responsive site with a top nav menu that shrinks to a hamburger button. Bootstrap would have been great for this, but I wanted to both challenge myself and keep my web app lean and minimal. I ended up using a small bit a jquery to get the nav menu style I was looking for.

With an app like this there is plenty of room to expand. I basically modeled its current functionality off of how I would use a budget/spending app. I currently use a mobile app for such purposes. Other people would benefit from an integration of budgeting into the program which could fit in nicely. A way to record income could also be a useful expansion to the app. These and many more minor improvements are potential areas for me to build out the web app.
