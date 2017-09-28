---
layout: post
title:  "Rails and create-react-app - CORS Deployment"
date:   2017-09-28 00:19:38 +0000
---


In this post I discuss methods to deploy a Create React App with a Rails back end. I go into detail on how to do this using CORS and separately deployed front/back-ends.

Create React App makes starting a React application quick and easy. Run a command or two and you already have a working React app that has Webpack, Babel and ESLint setup. It is all set and ready to run on a node server...but you want to run a Rails back end. In this post and a following post, I am going to discuss some different options available for when you want to integrate a `create-react-app` front end with a Rails API. In this post I will go over how to host the React app separately from the Rails back end.

## Setting up the Back End

First we configure our Rails API to allow cross-origin resource sharing (CORS). The `rack-cors` gem allows easy configuration of CORS.

```
# Gemfile

gem 'rack-cors'
```

If you created your Rails app in API mode (with --api), you just need to uncomment the line in the Gemfile. Otherwise just add it in a non-grouped area in the Gemfile. After updating the Gemfile make sure to run `bundle`.

Next we need to configure rack-cors. If you used Rails API, there will be a cors.rb in /config/initializers.

```
# cors.rb

Rails.application.config.middleware.insert_before 0, Rack::Cors do
 allow do
   origins ['localhost:3000', 'my-react-site.netlify.com']
   resource '*',
     headers: :any,
     methods: [:get, :post, :put, :patch, :delete, :options, :head]
 end
end
```

In the cors.rb file uncomment the section shown above and fill in the origins section. This is effectively whitelisting the front end location, allowing it to access the Rails API endpoints.

If you did not create your Rails app as an API you can just add something similar to your /config/application.rb. See [the docs](https://github.com/cyu/rack-cors) for further guidance.

Then we can deploy the Rails app. In this example we deploy it to Heroku. See the standard Heroku Rails deploy [guide](https://devcenter.heroku.com/articles/getting-started-with-rails5#deploy-your-application-to-heroku) for further detail.

```
$ heroku create
$ git push heroku master

...
... https://some-words-58239.herokuapp.com/ deployed to Heroku
```

Our back end Rails API is now hosted and ready to support our front end.

## Setting up the Front End

In our React app we need to make sure the Javascript fetch requests are being made to this Rails backend address.

```
const API_URL = "https://some-words-58239.herokuapp.com/"
// ....

fetch(API_URL + 'current')
// ....
```

We can also deploy the Create React App to a separate Heroku server if you have the React App in a separate git repository. If you push your React app repo to a new Heroku app, Heroku will serve the application from a node server. I do not recommend this method if you are using the Heroku Free Dyno Servers. You will have two different dynos to spool up from sleep which can take a while.

Instead, I prefer to serve the React app as a static site. There are various ways to deploy to static sites. Many companies offer continuous deployment strategies and command line interfaces. In our example we will build the static site locally and use the Netlify CLI.

From the React app root folder we run `npm run build` or `yarn build`. This will compile our React app into a static site and put that in the /build directory. We can then deploy this directory to Netlify.

```
$ npm install netlify-cli -g
$ netlify deploy
```

The first command above is to install the Netlify CLI. Running `netlify deploy` will prompt what directory to upload (specify `./build`). Many companies offer static site hosting that is affordable. Others that I have tried that were easy to use include surge.sh and Amazon S3.

Now our front end React app is hosted on Netlify and uses our Rails back end which is hosted on Heroku. If your Heroku app is on a free dyno, this is a good opportunity to make use of a React loading spinner for when the dyno is waking from sleep!

In a follow up post I will be walking through an alternative: deploying a React/Rails app on a single Heroku server.

