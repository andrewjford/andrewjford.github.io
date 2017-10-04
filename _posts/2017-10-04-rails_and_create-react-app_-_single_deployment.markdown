---
layout: post
title:      "Rails and create-react-app - Single Deployment"
date:       2017-10-04 22:21:26 +0000
permalink:  rails_and_create-react-app_-_single_deployment
---


In this post I walkthrough how to deploy a Create React App and Rails API to Heroku. We will have our Rails API deploy to Heroku and then setup the create-react-app to build and serve from that Heroku server.

As an alternative, in [this](https://andrewjford.github.io/2017/09/28/rails_and_create-react-app_-_cors_deployment/) previous post I outlined how to deploy a create-react-app and Rails API separately from each other.

For a single deployment to Heroku, our strategy is to have Heroku run create-react-app's `react-scripts` which builds the React app. Then serve that static front-end from the same server that is running the Rails back-end.

To start we first need to make sure our create-react-app is included in our Rails application. In this example we have the React app in a folder called client, located in our root Rails directory.

![directory example](https://images.imgbox.com/8e/6c/w2167rhQ_o.png)

We then need to make sure our React app is configured correctly. Since we will be serving the React front-end from the same host and port as the back-end we need to set all our front-end fetch requests with relative URLs. For example our requests could be like `fetch("/api/current")` rather than:

```
const API_URL = process.env.RAILS_API_URL;

fetch("API_URL/current")
//...
```

In `/client/package.json` add the following line. This will allow us to use the relative URL API requests in development as well.

```
"proxy": "http://localhost:3001"
```

The proxy line tells the React app to default requests to localhost:3001 in development. So in development we would run a node server that serves the React front end by entering `yarn start` from the client directory. That will serve the React app on port 3000. Then we run the development Rails server using the command `rails s -p 3001` from the Rails root directory.

Running the React app on its own server in development is useful because it allows for faster feedback through create-react-app's configured hot reloading.

Now that our React app is setup to fetch properly in development and production we need to manage our deployment. First we need to tell Heroku to build the React app when the Rails app is deployed. Create-react-app builds the React app using its dependency `react-scripts`.

```
/client/package.json
{
  "name": "client",
  "version": "0.1.0",
  "private": true,
  "proxy": "http://localhost:3001",
  "dependencies": {
    "react": "^15.6.1",
    "react-dom": "^15.6.1",
    "react-scripts": "1.0.13",
    "victory": "^0.22.2"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

Make sure `react-scripts` is under `dependencies` rather than `devDependencies` in the `/client/package.json` file.

Then in the Rails root directory `/package.json`, we set the following:

```
/package.json
{
  "name": "railsapp",
  "engines": {
    "node": "8.2.1"
  },
  "scripts": {
    "build": "cd client && yarn install && yarn build && cd ..",
    "deploy": "cp -a client/build/. public/",
    "postinstall": "yarn build && yarn deploy && echo 'Client built!'"
  }
}
```

This is the package.json that Heroku will initially read, running the scripts we have added. When the postinstall script is triggered, it will run the build and deploy scripts. The build script uses create-react-app's `react-scripts` to build a production version of the React app in the `/client/build` directory. The deploy script copies that folder to the `/public/` directory where it is served.

From the root directory, create a new Heroku app with `heroku apps:create` if you have not already.

```
heroku apps:create
```

Now we also have to tell Heroku to use this package.json file even though we are deploying a Rails app. Using the Heroku CLI we run the following to set a buildpack order.

```
heroku buildpacks:add heroku/nodejs --index 1
heroku buildpacks:add heroku/ruby --index 2
```

This tells Heroku to first use the package.json in the root directory to build a node server. After it builds that server it will run the postinstall script we added, which builds our React app in the `/public` folder. After this completes it runs the Ruby buildpack and deploys the Rails app.

Finally, make a Procfile in the root folder for Heroku:

```
web: bundle exec rails s
```

Now when we `git push heroku master`, Heroku will first build the React app using our script settings, then deploy and run the Rails server. The React app will be served at the root host URL.

This method of deploying has similarities to the CORS deployment I outlined in my previous post. They both build a static React app with the build scripts that are included in create-react-app. In this case we avoid CORS and instead serve the static site on the same host of the back-end server, utilizing Heroku buildpacks to automate the deployment process.

