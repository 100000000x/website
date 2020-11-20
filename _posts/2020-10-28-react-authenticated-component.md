---
title: React Authenticated Component
categories: 
    - React
    - Frontend
    - Boilerplate
author: Piotr Płoński
date: 2020-10-28
type: blog
---

In the [previous article](/tutorial/react-token-based-authentication-django) we've added signup and login features to the frontend. After login we have a redirect to the `Dashboard` view. However, you can access `Dashboard` view even if you are not logged in. In this post we will make the `AuthenticatedComponent` for `Dashboard` so only logged users will be able to access it. If not logged user would like to access the `Dashboard` URL then she will be redirected to `Login` with redirect in the URL `/login?next=/dashboard` after successful login. We will also add logout feature in the `Dashboard`.



## Authenticated Component

In `frontend/src/utils` directory please add a new file `RequireAuth.js`:

```jsx
// frontend/src/utils/RequireAuth.js
import React from "react";
import { connect } from "react-redux";
import { push } from "connected-react-router";
import PropTypes from "prop-types";

export default function requireAuth(Component) {
  class AuthenticatedComponent extends React.Component {
    constructor(props) {
      super(props);
      this.checkAuth();
    }

    componentDidUpdate(prevProps, prevState) {
      this.checkAuth();
    }

    checkAuth() {
      if (!this.props.isAuthenticated) {
        const redirectAfterLogin = this.props.location.pathname;
        this.props.dispatch(push(`/login?next=${redirectAfterLogin}`));
      }
    }

    render() {
      return (
        <div>
          {this.props.isAuthenticated === true ? (
            <Component {...this.props} />
          ) : null}
        </div>
      );
    }
  }
  AuthenticatedComponent.propTypes = {
    isAuthenticated: PropTypes.bool.isRequired,
    location: PropTypes.shape({
      pathname: PropTypes.string.isRequired
    }).isRequired,
    dispatch: PropTypes.func.isRequired
  };

  const mapStateToProps = state => {
    return {
      isAuthenticated: state.auth.isAuthenticated,
      token: state.auth.token
    };
  };

  return connect(mapStateToProps)(AuthenticatedComponent);
}

```

The `requireAuth()` function returns the `AuthenticatedComponent` that can wrap any component. It checks `isAuthenticated` variable from the `auth` store. If the user is authenticated then wrapped component is rendered. Otherwise, there is redirect to the `Login` view with `next` view coded in the URL:

```jsx
// read the current location
const redirectAfterLogin = this.props.location.pathname;
// go to login and pass current location in next parameter
this.props.dispatch(push(`/login?next=${redirectAfterLogin}`));
```
We check if the user is authenticated in the `constructor` and after the component update in `componentDidUpdate()`. It might be worth checking the [diagram showing React component lifecycle](https://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/). 

Please go to `frontend/src/App.js` file, and wrap `Dashboard` component with `requireAuth()` function:

```jsx
// frontend/src/App.js
// ...
// add import
import requireAuth from "./utils/RequireAuth";
// ...
// add requireAuth function
<Route path="/dashboard" component={requireAuth(Dashboard)} />
//...
```

Please clear your `localStorage` data if you are logged in (in the developer tools in the browser). Then, please open [`http://localhost:3000/dashboard`](http://localhost:3000/dashboard) in your browser. You should be redirected to [`http://localhost:3000/login?next=/dashboard`](http://localhost:3000/login?next=/dashboard). Please login, and you should be redirected to the `Dashboard` view.

## Add Navbar and logout in the Dashboard

Let's improve our `Dashboard`:
- We will add `Navbar` at the top of the website. We will but there link to `Home` view, information about username, and logout link.
- On logout click, the user will be redirected to `Home` view. All user information from `localStorage` will be removed.


```jsx
// frontend/src/components/dashboard/Dashboard.js

import React, { Component } from "react";
import PropTypes from "prop-types";
import { connect } from "react-redux";
import { withRouter } from "react-router-dom";

import { Container, Navbar, Nav } from "react-bootstrap";
import { logout } from "../login/LoginActions";

class Dashboard extends Component {
  onLogout = () => {
    this.props.logout();
  };

  render() {
    const { user } = this.props.auth;
    return (
      <div>
        <Navbar bg="light">
          <Navbar.Brand href="/">Home</Navbar.Brand>
          <Navbar.Toggle />
          <Navbar.Collapse className="justify-content-end">
            <Navbar.Text>
              User: <b>{user.username}</b>
            </Navbar.Text>
            <Nav.Link onClick={this.onLogout}>Logout</Nav.Link>
          </Navbar.Collapse>
        </Navbar>
        <Container>
          <h1>Dashboard</h1>
        </Container>
      </div>
    );
  }
}

Dashboard.propTypes = {
  logout: PropTypes.func.isRequired,
  auth: PropTypes.object.isRequired
};

const mapStateToProps = state => ({
  auth: state.auth
});

export default connect(mapStateToProps, {
  logout
})(withRouter(Dashboard));
```

The `logout` function is imported from `LoginActions.js` file. It is sending `POST` request to logout. What is more, it clears `user` and `token` information in the store and `localStorage`.

You should get the view as in the image below. Please play it with it a little. For example, you can try to login then open new tab and open [`http://localhost:3000/dashboard`](http://localhost:3000/dashboard) URL.

[![Dashboard with navbar](dashboard_navbar.png){:.image-border}](dashboard_navbar.png)

### Commit changes to the repository

Remember to commit changes to the repository:

```bash
# run from the main directory
git add frontend/src/utils/RequireAuth.js
git commit -am "add auth component"
git push
```

## Summary

- We created `AuthenticatedComponent` which check if user is logged in.
- If user is logged, then the wrapped component is rendered. Otherwise, there is a redirect to `Login` view.
- We added simple navbar in the `Dashboard` view. There is a `Logout` link, that sends logout request, clears all user data in the browser, and redirects to `Home` view.

## What's next?

In the next article we will add simple CRUD (Create, Read, Update, Delete) model in backend and frontend.

Next article: [CRUD in Django Rest Framework and React](/tutorial/crud-django-rest-framework-react).