---
layout: post
title:      "JWT Logins in React/Redux"
date:       2017-10-11 18:54:57 +0000
permalink:  jwt_logins_in_react_redux
---


In this post I discuss how to manage JSON web tokens in a React front end application that uses Redux. We will assume that the front end is making a call to an API back end that responds with a JWT. If you want detail on setting up JWT on a Rails API back end, see [this](https://andrewjford.github.io/2017/09/20/adding_jwt_authentication_to_a_rails_api/) post.

Also I am using `redux-thunk` middleware to allow asynch actions. For more information on using `redux-thunk` see [here](https://github.com/gaearon/redux-thunk).

First we will create a Login component to accept the user input.

```
//Login.js

import React from 'react';
import { connect } from 'react-redux';
import { bindActionCreators } from 'redux';

import FlashMessage from './FlashMessage';
import {
  changeEmailInput,
  changePasswordInput,
  clearLoginInput,
  loginUser
} from '../actions/sessionActions';

class Login extends React.Component {

  handleSubmit = (event) => {
    event.preventDefault();

    let formInput = {
      email: this.props.session.input.email,
      password: this.props.session.input.password
    }

    this.props.loginUser(formInput);
  }

  handleEmailChange = (event) => {
    this.props.changeEmailInput(event.target.value);
  }

  handlePasswordChange = (event) => {
    this.props.changePasswordInput(event.target.value);
  }

  render() {
    return <div>
      <h2>Login</h2>

      <form className="flex-vertical" onSubmit={this.handleSubmit}>
        <input type="text"
          placeholder="Email"
          value={this.props.session.input.email}
          onChange={this.handleEmailChange}/>

        <input type="password"
          placeholder="Password"
          value={this.props.session.input.password}
          onChange={this.handlePasswordChange}/>

        <button type="submit" className="center-button">
          Login
        </button>
      </form>

    </div>
  }
}

const mapStateToProps = (state) => {
  return {
    session: state.session
  }
}

const mapDispatchToProps = (dispatch) => {
  return bindActionCreators({
    changeEmailInput: changeEmailInput,
    changePasswordInput: changePasswordInput,
    clearLoginInput: clearLoginInput,
    loginUser: loginUser,
  }, dispatch)
}

export default connect(mapStateToProps, mapDispatchToProps)(Login);
```

This component renders a form with email and password input. The inputs are controlled through Redux. It has several handlers for the changes in input as well as the form submittal. These handlers use the Redux actions mapped from the sessionActions file. When the form is submitted, the values of the inputs are passed into the loginUser action.

Before we jump to the actions file, we detour to a service object to handle the actual fetch requests.

```
// SessionService.js

class SessionApi {
  static login(credentials) {
    const request = new Request(API_URL+'login', {
      method: 'POST',
      headers: new Headers({
        'Content-Type': 'application/json'
      }),
      body: JSON.stringify({auth: credentials})
    });

    return fetch(request)
      .then(response => response.json())
      .catch(error => {
        return error;
      });
  }
}

export default SessionApi;
```

This service object will make the fetch requests to the back end API. For logging in a user, we build out a `new Request` to specify the method and body. In the body we JSON.stringify the email and password in the format our API is expecting; in this case the back end expects the credentials under an 'auth' key. We then use this request object with fetch. Calling the login function will return the fetch promise.

```
// sessionActions.js

import SessionApi from '../services/SessionService';

export function loginUser(credentials) {
  return function(dispatch) {
    return SessionApi.login(credentials)
      .then(response => {
        if(response.jwt){
          sessionStorage.setItem('jwt', response.jwt);
          dispatch(loginSuccess());
        }
        else {
          dispatch(loginFailure());
        }
      })
  }
}

export function loginSuccess() {
  return {
    type: 'LOGIN_SUCCESS'
  }
}

export function changePasswordInput(newInput) {
  return {
    type: 'CHANGE_PASSWORD_INPUT',
    payload: newInput,
  }
}

export function changeEmailInput(newInput) {
  return {
    type: 'CHANGE_EMAIL_INPUT',
    payload: newInput,
  }
}
```

In the actions file, we have some standard actions to update the email and password inputs. We also have the loginUser action. The loginUser action passes its parameters (our form input) to the SessionApi.login() function we previously built (and imported at the top of this file).

If the response has a JWT, we store the token in session storage under the key `jwt` with `sessionStorage.setItem('jwt', response.jwt);` and dispatch the loginSuccess action.

```
function sessionReducer(
  state = {session: !!sessionStorage.jwt,
    input: {email: "", password: ""},
  }, action) {

  switch(action.type){
    case "CHANGE_EMAIL_INPUT":
      return {...state, input: {...state.input, email: action.payload}}
    case "CHANGE_PASSWORD_INPUT":
      return {...state, input: {...state.input, password: action.payload}}
    case "CLEAR_LOGIN_INPUT":
      return {...state, input: {email: "", password: ""}}
    case "LOGIN_SUCCESS":
      return {...state, input: {email: "", password: ""}, session: true}
    default:
      return state;
  }
}

export default sessionReducer;
```

Our reducer file has state for sessionform input. The session.state is just a boolean for whether the user has logged in. With LOGIN_SUCCESS, the form input is cleared and state.session is set to true. We can then use this boolean in the rest of our application to check whether the user is logged in, while the token itself can accessed from sessionStorage.

So in review, we setup a service object to make the API calls to the back end. Then using `redux-thunk` we have a Login action that will handle the response. If the response is successful and we receive a token, the token is stored in Session Storage and we update our related boolean in state.

