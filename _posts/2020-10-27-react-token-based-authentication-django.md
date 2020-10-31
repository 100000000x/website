---
title: React Token Based Authentication to Django Backend
categories: 
 - React
 - Token based authentication
 - Frontend
author: Piotr Płoński
date: 2020-10-27
type: blog
---

In this post, we will write user interface in React for token-based authentication to Django backend. We will use code from previous post: [Token Based Authenitcation with Django Rest Framework and Djoser](/blog/token-based-authentication-django-rest-framework-djoser) (code with tag [v2](https://github.com/saasitive/django-react-boilerplate/tree/v2))

This post will be splitted into following parts:

- First create React components (Home, Signup, Login, Dashboard).
- Create React routing with `react-router-dom` package.
- Add Redux to React.
- Add Signup actions and reducer.
- Add Login actions and reducer.
- Create `AuthenticatedComponent` for authenticated routing.
- Add CORS and CSRF protections to Django server. It will be required because we will make requests from other port than server and to be safe. The CSRF token protect server agains [Cross-Site Request Forgery](https://en.wikipedia.org/wiki/Cross-site_request_forgery).

In this tutorial we will use [`axios`](https://github.com/axios/axios) package for doing server requests from React. We will use [`bootstrap`](https://getbootstrap.com/) with [`react-bootstrap`](https://react-bootstrap.github.io/) package to build frontend user internaface.

If you would like to see post hto to use React with other than `bootstrap` packages, please let me know by filling the [form](https://forms.gle/rgAG9gkhUEH2wUVt5). Other packages can be: [Material](https://material-ui.com/), [Ant](https://ant.design/docs/react/introduce), [Bulma](https://bulma.io/), or [Tailwind](https://tailwindcss.com/).

## Add React Components

Please open a new terminal window and navigate to `frontend` directory. Then, please run the development server:

```bash
npm start
```

It has hot-reloading, which means that all changes (after file save) are forcing the server to reload and serve the newest version (so we don't have to hit F5 constantly). You should see your frontend running at [http://localhost:3000/](http://localhost:3000/).

### Install packages

Please stop the dev server for a moment. We will need to install few new packages for start:

```bash
# for user interface
npm install react-bootstrap bootstrap
# for routing
npm install react-router-dom
```

The `react-router-dom` [documentation](https://reactrouter.com/web/guides/quick-start) (if you need to check it). After installation please run again the dev server (`npm start`). You can also check the `frontend/package.json` file, the newly installed packages should be added there.

You need to add `bootstrap.css` import in `frontend/src/index.js` file.

```jsx
// ...
// add this import in frontend/src/index.js
// add it before index.css import
import "bootstrap/dist/css/bootstrap.css";
// ...
```

### Let's first clean our React app.

Please remove the `frontend/src/App.css` file and remove all content in `frontend/src/index.css` (we will keep the file, maybe it will be needed in the future). Please also delete files:

- `frontend/public/logo192.png`
- `frontend/public/logo512.png`
- `frontend/src/logo.svg`

Then, please update the `frontend/public/manifest.json` file:

```json
{
  "short_name": "SaaSitive",
  "name": "SaaSitive Django+React Boilerplate",
  "icons": [
    {
      "src": "favicon.ico",
      "sizes": "64x64 32x32 24x24 16x16",
      "type": "image/x-icon"
    }
  ],
  "start_url": ".",
  "display": "standalone",
  "theme_color": "#000000",
  "background_color": "#ffffff"
}
```

You can use your values for `short_name` and `name`. 

Let's go to `frontend/src/App.js`, remove its content, and replace it with the following code:

```jsx
import React, { Component } from "react";

class App extends Component {
  render() {
    return <h1>Hi!</h1>;
  }
}

export default App;
```

The development server should refresh the [`http://localhost:3000`](http://localhost:3000) website and show "Hi!".

OK, let's add `components` directory in `frontend/src` and `Home.js` file in it.

```jsx
// frontent/src/components/Home.js
import React, { Component } from "react";
import { Container } from "reactstrap";

class Home extends Component {
  render() {
    return (
      <Container>
        <h1>Home</h1>
      </Container>
    );
  }
}

export default Home;
```

Wait ... How to navigate to `Home.js` view? We will need a router! (It is already installed from `react-router-dom` package)

Let's go to `App.js` file.

```jsx
// frontend/src/App.js
import React, { Component } from "react";
import { BrowserRouter, Route, Switch } from "react-router-dom";
import Home from "./components/Home";

class App extends Component {
  render() {
    return (
      <div>
        <BrowserRouter>
          <Switch>
            <Route exact path="/" component={Home} />
          </Switch>
        </BrowserRouter>
      </div>
    );
  }
}

export default App;
```

We added `BrowsableRouter` with `Switch` and one `Route` to `path="/"` with route to our `Home` component. We can add more components. We will add them in separate directories, because later we will create actions and reducers for each (the Redux part).

Let's add `signup`, `login`, `dashboard` directories in `frontend/src/components`. 

Please add `Signup` component in the file `frontend/src/componenets/signup/Signup.js`:

```jsx
// frontend/src/componenets/signup/Signup.js

import React, { Component } from "react";
import { Container } from "react-bootstrap";

class Signup extends Component {
  render() {
    return (
      <Container>
        <h1>Signup</h1>
      </Container>
    );
  }
}

export default Signup;
```

Please add `Login` component in the file `frontend/src/componenets/login/Login.js`:

```jsx
// frontend/src/componenets/login/Login.js

import React, { Component } from "react";
import { Container } from "react-bootstrap";

class Login extends Component {
  render() {
    return (
      <Container>
        <h1>Login</h1>
      </Container>
    );
  }
}

export default Login;
```


Please add `Dashboard` component in the file `frontend/src/componenets/dashboard/Dashboard.js`:

```jsx
// frontend/src/componenets/dashboard/Dashboard.js

import React, { Component } from "react";
import { Container } from "react-bootstrap";

class Dashboard extends Component {
  render() {
    return (
      <Container>
        <h1>Dashboard</h1>
      </Container>
    );
  }
}

export default Dashboard;
```

We have three new components which we will add to our router. We won't be able to access them without adding them to the router

Please update the `frontend/src/App.js` file:

```jsx
import React, { Component } from "react";
import { BrowserRouter, Route, Switch } from "react-router-dom";
import Home from "./components/Home";
import Signup from "./components/signup/Signup";
import Login from "./components/login/Login";
import Dashboard from "./components/dashboard/Dashboard";

class App extends Component {
  render() {
    return (
      <div>
        <BrowserRouter>
          <Switch>
            <Route path="/signup" component={Signup} />
            <Route path="/login" component={Login} />
            <Route path="/dashboard" component={Dashboard} />
            <Route exact path="/" component={Home} />
          </Switch>
        </BrowserRouter>
      </div>
    );
  }
}

export default App;
```

OK, it is time to test it! You can enter in the browser different paths and check if correct components are displayed. For example, please enter [`http://localhost:3000/login`](http://localhost:3000/login) and you should see the `Login` component. The  [`http://localhost:3000/dashboard`](http://localhost:3000/dashboard) should show a `Dashboard` component.

Maybe you wonder what is the difference between `Route path` and `Route exact path` (used for `Home`). You can try to check it yourself:

- please enter: [`http://localhost:3000/signup/bla/bla/bla`](http://localhost:3000/signup/bla/bla/bla)
- please enter: [`http://localhost:3000/bla/bla/bla`](http://localhost:3000/bla/bla/bla)

The first one will show you `Signup` component, while the second one will show a blank window. The first URL will match the `Signup` route. The second one won't have match. If you would like to have a default match for any URL you can add the default route:

```jsx
<Route path="*">Ups</Route>
```

It will display "Ups" for unknown matches.

Your fronted code structure should look like below after this part of the post:

```bash
.
├── package.json
├── package-lock.json
├── public
│   ├── favicon.ico
│   ├── index.html
│   ├── manifest.json
│   └── robots.txt
├── README.md
├── src
│   ├── App.js
│   ├── App.test.js
│   ├── components
│   │   ├── dashboard
│   │   │   └── Dashboard.js
│   │   ├── Home.js
│   │   ├── login
│   │   │   └── Login.js
│   │   └── signup
│   │       └── Signup.js
│   ├── index.css
│   ├── index.js
│   ├── reportWebVitals.js
│   └── setupTests.js
└── yarn.lock
```

### React components

Let's add more code to the components. We will make `Home.js` as the main view with links to others:

```jsx
// frontend/src/components/Home.js

import React, { Component } from "react";
import { Link } from "react-router-dom";
import { Container } from "react-bootstrap";

class Home extends Component {
  render() {
    return (
      <Container>
        <h1>Home</h1>
        <p>
          <Link to="/login/">Login</Link>
        </p>
        <p>
          <Link to="/signup">Sign up</Link>
        </p>
        <p>
          <Link to="/dashboard">Dashboard</Link>
        </p>
      </Container>
    );
  }
}

export default Home;
```

Please take a look that we are using `Link` from `react-router-dom` package for creating links. These links will use router to navigate to the correct view.

Now, we will define the user interface for creating the user account. Let's add more code to `Signup.js` file.

```jsx
// frontend/src/components/signup/Signup.js

import React, { Component } from "react";
import { Link } from "react-router-dom";
import {
  Container,
  Button,
  Row,
  Col,
  Form,
  FormControl
} from "react-bootstrap";

class Signup extends Component {
  render() {
    return (
      <Container>
        <Row>
          <Col md="4">
            <h1>Signup</h1>
            <Form>
              <Form.Group controlId="usernameId">
                <Form.Label>User name</Form.Label>
                <Form.Control
                  type="text"
                  name="username"
                  placeholder="Enter user name"
                />
                <FormControl.Feedback type="invalid"></FormControl.Feedback>
              </Form.Group>

              <Form.Group controlId="passwordId">
                <Form.Label>Your password</Form.Label>
                <Form.Control
                  type="password"
                  name="password"
                  placeholder="Enter password"
                />
                <Form.Control.Feedback type="invalid"></Form.Control.Feedback>
              </Form.Group>
            </Form>
            <Button color="primary">Sign up</Button>
            <p className="mt-2">
              Already have account? <Link to="/login">Login</Link>
            </p>
          </Col>
        </Row>
      </Container>
    );
  }
}

export default Signup;
```

We use `bootstrap` forms to create signup form. It is only a visual part of it. Is is not going to perform any actions, but we will add it soon (in this post). Please take a notice that only username and password is requred to create the account. In the future post, there will be an example where there will be needed: user name, email address, password and repeated password (the login will be based on email + password).  

The code for `Login.js` (which is very similar to `Signup.js`):

```jsx
// frontend/src/components/login/Login.js

import React, { Component } from "react";
import { Link } from "react-router-dom";
import {
  Container,
  Button,
  Row,
  Col,
  Form,
  FormControl
} from "react-bootstrap";

class Login extends Component {
  render() {
    return (
      <Container>
        <Row>
          <Col md="4">
            <h1>Login</h1>
            <Form>
              <Form.Group controlId="usernameId">
                <Form.Label>User name</Form.Label>
                <Form.Control
                  type="text"
                  name="username"
                  placeholder="Enter user name"
                />
                <FormControl.Feedback type="invalid"></FormControl.Feedback>
              </Form.Group>

              <Form.Group controlId="passwordId">
                <Form.Label>Your password</Form.Label>
                <Form.Control
                  type="password"
                  name="password"
                  placeholder="Enter password"
                />
                <Form.Control.Feedback type="invalid"></Form.Control.Feedback>
              </Form.Group>
            </Form>
            <Button color="primary">Login</Button>
            <p className="mt-2">
              Don't have account? <Link to="/signup">Signup</Link>
            </p>
          </Col>
        </Row>
      </Container>
    );
  }
}

export default Login;
```

After above steps you should have following views:

![](home.png){:.image-border}

![](signup.png){:.image-border}

![](login.png){:.image-border}

![](dashboard.png){:.image-border}

## Add Redux to React

[Redux](https://redux.js.org/) will help as organize and manage application data. It consists of three main concepts:

- **Store** which keeps information about application state (data),
- **Reducer** that specify how the state is changed in response to actions,
- **Actions** the source of information for the **Store**.

Before adding new code, let's install necessary packages:

```bash
npm install redux react-redux redux-thunk connected-react-router
```

First we will define **Reducer**. Please add `Reducer.js` file in `frontend/src` directory:

```jsx
// frontend/src/Reducer.js

import { combineReducers } from "redux";
import { connectRouter } from "connected-react-router";

const createRootReducer = history =>
  combineReducers({
    router: connectRouter(history)
  });

export default createRootReducer;
```

If we will create a new reducer for a component we will add it here.

Then, we need to add **Store** to our application. Please add `Root.js` file in `frontend/src` directory:

```jsx
// frontend/src/Root.js

import React from "react";
import thunk from "redux-thunk";
import { Provider } from "react-redux";
import { createBrowserHistory } from "history";
import { applyMiddleware, createStore } from "redux";
import { routerMiddleware, ConnectedRouter } from "connected-react-router";

import rootReducer from "./Reducer";

const Root = ({ children, initialState = {} }) => {
  const history = createBrowserHistory();
  const middleware = [thunk, routerMiddleware(history)];

  const store = createStore(
    rootReducer(history),
    initialState,
    applyMiddleware(...middleware)
  );

  return (
    <Provider store={store}>
      <ConnectedRouter history={history}>{children}</ConnectedRouter>
    </Provider>
  );
};

export default Root;
```

The last change will replace `Router` with `Root` component in `App.js`.

```jsx
// frontend/src/App.js

import React, { Component } from "react";
import Root from "./Root"; // <------------- new import
import { Route, Switch } from "react-router-dom"; // <--- remove Router
import Home from "./components/Home";
import Signup from "./components/signup/Signup";
import Login from "./components/login/Login";
import Dashboard from "./components/dashboard/Dashboard";

class App extends Component {
  render() {
    return (
      <div>
        <Root> {/* replace Router with Root */}
          <Switch>
            <Route path="/signup" component={Signup} />
            <Route path="/login" component={Login} />
            <Route path="/dashboard" component={Dashboard} />
            <Route exact path="/" component={Home} />
            <Route path="*">Ups</Route>
          </Switch>
        </Root> {/* replace Router with Root */}
      </div>
    );
  }
}

export default App;
```

The application should run without any changes in behavior. We've just added empty skeleton. This is the time to add the first actions and reducer to create a new user (sign up view)!

Let's add new files in `frontend/src/components/signup` directory:

- `SignupTypes.js` - it will define types of actions in the sign up component,
- `SignupActions.js` - it will implements actions needed for sign up,
- `SignupReducer.js` - it will implements how actions change the sign up data store.

The content of `SignupTypes.js`:

```jsx
// frontend/src/components/signup/SignupTypes.js
export const CREATE_USER_SUBMITTED = "CREATE_USER_SUBMITTED";
export const CREATE_USER_SUCCESS = "CREATE_USER_SUCCESS";
export const CREATE_USER_ERROR = "CREATE_USER_ERROR";
```

There are three types of actions:

- `CREATE_USER_SUBMITTED`, which means that the request to create user was send and we are waiting for server response. After this action we should disable `Signup` button till there is server response.
- `CREATE_USER_SUCCESS` - the action is called after `HTTP 201 CREATED` response from server, which means that user was created.
- `CREATE_USER_ERROR` - the action is called when there was an error during creation of the user and it was not created.

We have types of actions defined, so let's create the **Reducer**. Please add the following content to `SignupReducer.js`:

```jsx
// frontend/src/components/SignupReducer.js

// import needed actions
import {
  CREATE_USER_ERROR,
  CREATE_USER_SUBMITTED,
  CREATE_USER_SUCCESS
} from "./SignupTypes";

// define the initial state of the signup store
const initialState = {
  usernameIsInvalid: false,
  usernameError: "",
  passwordIsInvalid: false,
  passwordError: "",
  isSubimtted: false
};

// define how action will change the state of the store
export const signupReducer = (state = initialState, action) => {
  switch (action.type) {
    case CREATE_USER_SUBMITTED:
      return {
        usernameIsInvalid: false,
        usernameError: "",
        passwordIsInvalid: false,
        passwordError: "",
        isSubimtted: true
      };
    case CREATE_USER_ERROR:
      const errorState = {
        usernameIsInvalid: false,
        usernameError: "",
        passwordIsValid: true,
        passwordError: "",
        isSubimtted: false
      };
      if (action.errorData.hasOwnProperty("username")) {
        errorState.usernameIsInvalid = true;
        errorState.usernameError = action.errorData["username"];
      }
      if (action.errorData.hasOwnProperty("password")) {
        errorState.passwordIsInvalid = false;
        errorState.passwordError = action.errorData["password"];
      }
      return errorState;
    case CREATE_USER_SUCCESS:
      return {
        usernameIsInvalid: false,
        usernameError: "",
        passwordIsInvalid: true,
        passwordError: "",
        isSubimtted: false
      };
    default:
      return state;
  }
}
```

The above **Reducer** accepts as input the `initialState`:

```jsx
const initialState = {
  usernameIsInvalid: false,
  usernameError: "",
  passwordIsInvalid: false,
  passwordError: "",
  isSubimtted: false
};
```
The `initialState` defines what attributes will be in our store. It also set the inital (default) values which are set at the beginning of application.

There is a big `switch` statement in the `signupReducer` with separate `case` for each action type. Each action type define its own way how to change the selected state of the application. Please remember to add `default` case which will just return the curtrent state. That's all, we have a reducer!

We need to add signup reducer to the root reducer in `frontend/src/Reducer.js`:

```jsx
// frontend/src/Reducer.js
import { combineReducers } from "redux";
import { connectRouter } from "connected-react-router";

// import new reducer
import { signupReducer } from "./components/signup/SignupReducer";

const createRootReducer = history =>
  combineReducers({
    router: connectRouter(history),
    createUser: signupReducer // add it here
  });

export default createRootReducer;
```

Before defining actions we need to install one more (maybe two) packages. We need a package for sending requests to the server. I'm using [`axios`](https://github.com/axios/axios) package.

One more package, that is optional, I like to add toast notifications. For this I'm using [`react-toastify`](https://github.com/fkhadra/react-toastify). It is optional and you can add different way to display notifications.

The package installation command:

```bash
# should run in frontend directory
npm install axios react-toastify
```

Let's add action to send the request to the server to create a new user:


```jsx
// frontend/src/components/signup/SignupActions.js

import axios from "axios";
import { push } from "connected-react-router";
import { toast } from "react-toastify";
import { isEmpty } from "../../utils/Utils";
import {
  CREATE_USER_ERROR,
  CREATE_USER_SUBMITTED,
  CREATE_USER_SUCCESS
} from "./SignupTypes";

export const createUser = userData => dispatch => {
  dispatch({ type: CREATE_USER_SUBMITTED }); // set submitted state
  axios
    .post("/api/v1/users/", userData)
    .then(response => {
      toast.success(
        "Account for " +
          userData.username +
          " created successfully. Please login."
      );
      dispatch({ type: CREATE_USER_SUCCESS });
    })
    .catch(error => {
      if (
        !isEmpty(error) &&
        !isEmpty(error.response) &&
        error.response.hasOwnProperty("data")
      ) {
        // known error
        toast.error(JSON.stringify(error.response.data));
        dispatch({
          type: CREATE_USER_ERROR,
          errorData: error.response.data
        });
      } else {
        if (!isEmpty(error.message)) {
          toast.error(JSON.stringify(error.message));
        } else {
          toast.error(JSON.stringify(error));
        }
      }
    });
};
```

There is only one action in the `SignupActions.js`. It send `POST` request to server (at url `/api/v1/users/`) with `userData`. It aslo defines what to do next:

- if success then green toast is displayed and `CREATE_USER_SUCCESS` is dispatched,
- in the case of error it dsipaches `CREATE_USER_ERROR` and show red toast.

Is it going to work?

Not yet.

We still need to add few things:

- We need to connect signup action with `Signup` button - when we click the button the request should be send.
- We need to collect data from signup form to be able to create `userData` JSON.
- We need to configure `axios` - we need to set the server URL.
- The last thing, we need to set CORS in the server.
