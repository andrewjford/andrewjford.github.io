---
layout: post
title:      "Modals in React/Redux"
date:       2017-10-25 18:37:44 +0000
permalink:  modals_in_react_redux
---


Recently I have been working on adding login and signup modals to one of my React/Redux applications. These modals would be accessed from the navbar, but I was unsure of how they should be structured in a React and Redux design. In this post I will outline a common approach for handling modals in a React/Redux application.

The general overview of this approach is to have a single modal container which displays modals based on what you have in your Redux store. Components that trigger a modal, such as a link in the navbar, would dispatch actions to the modal store. This centralizes all the modals in the web app which I felt was a clean way to organize and structure modals.

In this example I will use a login modal activated from the navbar.

```
// Navbar.js
import React from 'react';
import { Link, NavLink } from 'react-router-dom';
import { connect } from 'react-redux';
import { bindActionCreators } from 'redux';

import { openLoginWindow } from '../actions/modalActions';

class Navbar extends React.Component {

  handleLoginClick = (event) => {
    this.props.openLoginWindow();
  }

  render() {
    return <nav>
        <NavLink
          to="/about"
          >About</NavLink>

        <Link
          onClick={this.handleLoginClick}
          to={`/`}
          >Login</Link>
      </nav>
  }
}

const mapDispatchToProps = (dispatch) => {
  return bindActionCreators({
    openLoginWindow: openLoginWindow,
  }, dispatch)
}

export default connect(null, mapDispatchToProps)(Navbar);
```

Here we connect our Navbar to be able to dispatch an `openLoginWindow` action in modalActions.

```
// modalActions.js
export function openLoginWindow() {
  return {
    type: 'OPEN_LOGIN_WINDOW'
  }
}

export function closeLoginWindow() {
  return {
    type: 'CLOSE_LOGIN_WINDOW'
  }
}

export function openSignupWindow() {
  return {
    type: 'OPEN_SIGNUP_WINDOW'
  }
}

export function closeSignupWindow() {
  return {
    type: 'CLOSE_SIGNUP_WINDOW'
  }
}
```

We create an actions file called modalActions. Here we have actions to open and close the Login modal and the Signup modal. The actions are used in the modalReducer.

```
// modalReducer.js
function modalReducer(
  state = {
    loginOpen: false,
    signupOpen: false,
  }, action) {

  switch(action.type){
    case "OPEN_LOGIN_WINDOW":
      return {...state, loginOpen: true}
    case "CLOSE_LOGIN_WINDOW":
      return {...state, loginOpen: false}
    case "OPEN_SIGNUP_WINDOW":
      return {...state, signupOpen: true}
    case "CLOSE_SIGNUP_WINDOW":
      return {...state, signupOpen: false}
    default:
      return state;
  }
}

export default modalReducer;
```

The reducer that takes in the modal actions stores a boolean for whether a modal should be open or not.

Don't forget to add the new modalReducer to your store. Below I add the modalReducer to the rootReducer in index.js.

```
// index.js
// lots of imports....

const rootReducer = combineReducers({
  map: mapReducer,
  user: userReducer,
  session: sessionReducer,
  modal: modalReducer,
});

const store = createStore(
  rootReducer, composeWithDevTools(applyMiddleware(thunk),)
);

// other code....
```

We use a container to hold the modals. Since this is meant to be an app-wide modal container, it should be rendered in one of your main components. This container connects to state.modal in the store, and displays the modals based on those values.

```
// ModalContainer.js
import React from 'react';
import { connect } from 'react-redux';

import Login from '../components/Login';
import SignupModal from '../components/SignupModal';

class ModalContainer extends React.Component {
  render() {

    return <div>
      {this.props.modal.loginOpen ? <Login /> : null}
      {this.props.modal.signupOpen ? <SignupModal /> : null}
    </div>
  }
}

const mapStateToProps = (state) => {
  return {
    modal: state.modal
  }
}

export default connect(mapStateToProps)(ModalContainer);
```

The Signup component is set to display from the modal container when the modal store value `signupOpen` is `true`.

```
// Signup.js
import React from 'react';
import { connect } from 'react-redux';
import { bindActionCreators } from 'redux';

import SignupForm from './SignupForm';
import {
  openLoginWindow,
  closeSignupWindow,
} from '../actions/modalActions';
import { clearLoginInput } from '../actions/sessionActions';

class SignupModal extends React.Component {

  handleOutsideClick = (event) => {
    if(event.target.classList.contains("overlay-blanket")){
      this.props.closeSignupWindow();
      this.props.clearLoginInput();
    }
  }

  handleLoginLink = (event) => {
    event.preventDefault();
    this.props.openLoginWindow();
    this.props.closeSignupWindow();
  }

  render(){
    return <div className="overlay-blanket" onClick={this.handleOutsideClick}>
      <div className="center-overlay">
        <div className="center-relative">
          <h2>Signup</h2>
          <SignupForm />
          <span>
            Already a member? <a href="" onClick={this.handleLoginLink}>Login</a>
          </span>
        </div>
      </div>
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
    openLoginWindow: openLoginWindow,
    closeSignupWindow: closeSignupWindow,
    clearLoginInput: clearLoginInput,
  }, dispatch)
}

export default connect(mapStateToProps, mapDispatchToProps)(SignupModal);

```

The modal component itself renders from the ModalContainer. In this example I implement simple modal functionality using my own CSS and Javascript implementation.

I recommend checking out `react-modal` for implementing the actual modal component itself. It has accessibility features handled correctly (unlike my simple example above). Accessibility with modals can be tricky and using `react-modal` helps ensure proper accessibility of your modal.

