---
layout: post
title:  "Adding JWT authentication to a Rails API"
date:   2017-09-20 20:39:12 +0000
---


Adding an authentication token system can be a bit confusing the first time around if you are only familiar with session based authentication. In this post I am going to walkthrough adding token authentication to a Rails API using JWT.

JWT (JSON Web Token) is an industry standard method for representing claims to be transmitted between parties. In the case of our Rails API, the back end will generate a JWT to issue to the client front end. The token will then be used to allow access to the API endpoints.

To start we will setup our gemfile and models. First we need to add JWT and bcrypt to our gemfile.

```
# gemfile

# Use ActiveModel has_secure_password
gem 'bcrypt', '~> 3.1.7'
# JWT for Auth
gem 'jwt'
```

Make sure to run bundle afterwards.

Since we don't have any user model yet I generate a model.
```
$ rails generate model User email:string password_digest:string
```
And in the user model add `has_secure_password`.

Continuing on with the JWT integration, we create a class Auth that will create and decode tokens using the 'jwt' library gem. I put it in the app/services folder.

```
class Auth
  ALGORITHM = 'HS256'
  @auth_secret = ENV['AUTH_SECRET']

  def self.issue(payload)
    JWT.encode(
      payload,
      @auth_secret,
      ALGORITHM
    )
  end

  def self.decode(token)
    JWT.decode(token,
      @auth_secret,
      true,
      { algorithm: ALGORITHM }).first
  end
end
```
ALGORITHM is the type of algorithm you want JWT to use, in this case we are using HS256 which is HMAC SHA-256 (the default). The @auth_secret value is a secret string we have stored in the environment. Running `rails secret` in the terminal is a good way of generating a secret value to use here.

The issue method takes a parameter and encodes it using the @auth_secret and ALGORITHM. It then returns a token. We will use this method when a user logs in, passing their id in as a parameter.

The decode method takes a token and decodes it into the related user id. We will use this to authenticate requests on our API endpoints.

Next we update the Application Controller.

```
class ApplicationController < ActionController::API
  before_action :authenticate

  def logged_in?
    !!current_user
  end

  def current_user
    @current_user ||= User.find(decoded_token["user"]) if auth_present?
  end

  def authenticate
    render json: {error: "unauthorized"}, status: 401 unless logged_in?
  end


  private

  def auth_present?
    !!request.env.fetch("HTTP_AUTHORIZATION","").scan(/Bearer/).flatten.first
  end

  def decoded_token
    Auth.decode(token)
  end

  def token
    request.env["HTTP_AUTHORIZATION"].scan(/Bearer (.*)$/).flatten.last
  end

end
```

We have a before_action that runs the method authenticate on any actions in our API. Calling the authenticate method will use the private methods to decode the token passed in the HTTP request using the Auth.decode method we created previously. It then attempts to find the user based on the decoded token. This restricts access to our API endpoints unless the request has a valid token.

We also need to setup the controllers to give the client the proper token at user login.

```
class SessionsController < ApplicationController
  skip_before_action :authenticate, only: [:create]

  def create
    user = User.find_by(email: auth_params[:email])
    if user && user.authenticate(auth_params[:password])
      jwt = Auth.issue({user: user.id})
      render json: {jwt: jwt}
    else
      render json: {errors: {"Improper credentials": "Invalid username or password"}}, status: 401
    end
  end

  private
  def auth_params
    params.require(:auth).permit(:email, :password)
  end

end
```

We create a sessions controller to handle logging a user in and issuing a token. Note that we have a `skip_before_action` here relating to the create action. This is so we do not try to authenticate the token on this action. Like most session#create actions, we accept the user login credentials (in this case email and password) through strong parameters to sanitize our input. 

We then find the user by email and call the authenticate method on the user, passing in the given password. This uses the bcrypt gem we added at the beginning of this walkthrough to check the given password against the password_digest saved in the database. If the user email and password are correct, we run the Auth.issue method to create a token based on the user.id and send that token back to the client.

Finally we need to create a route for this sessions controller action.

```
Rails.application.routes.draw do
	# other routes...

  post '/login', to: 'sessions#create'

end

```

We expose a /login route that the client can post a user and password to, in order to login and receive a jwt token. 

