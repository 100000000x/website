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
- Add CORS and CSRF protections to Django server. It will be required because we will make requests from other port than server.

In this tutorial we will use [`axios`](https://github.com/axios/axios) package for doing server requests. For frontend user internaface we will use [`bootstrap`](https://getbootstrap.com/) with [`react-bootstrap`](https://react-bootstrap.github.io/) package.

If you would like to see post with other than `bootstrap` package, please let me know by filling the [form](https://forms.gle/rgAG9gkhUEH2wUVt5).

## Add React Components

Please open new terminal window and navigate to `frontend` directory. Please run the development server:

```bash
npm start
```

It has hot-reloading, which means that all changes (after file save) are forcing the server to reload and serve the newest version (so we don't have to hit F5 constantly). You should see your frontend running at [http://localhost:3000/](http://localhost:3000/).

### Install packages

Please stop the dev server for a moment. We will need to install few packages for start:

```bash
# for user interface
npm install react-bootstrap bootstrap
# for routing
npm install react-router-dom
```

The `react-router-dom` [documentation](https://reactrouter.com/web/guides/quick-start) (if you need to check it). After installation please run again the dev server (`npm start`). You can also check the `frontend/package.json` file, the newly installed packages should be added there.

You need to add `bootstrap.css` import in `frontend/src/index.js` file.

```js
// ...
// add this import in frontend/src/index.js
import "bootstrap/dist/css/bootstrap.css";
// ...
```

### Let's first clean our React app.

Please remove the `frontend/src/App.css` file and please remove all content in `frontend/src/index.css` (we will keep the file, maybe will be needed in the future). Please also delete:

- `frontend/public/logo192.png`
- `frontend/public/logo512.png`
- `frontend/src/logo.svg`

Please update the `frontend/public/manifest.json` file:

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

You can use any values for `short_name` and `name`. 

Let's go to `frontend/src/App.js`, remove its content, and replace it with the following code:

```js
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

```js
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

How to navigate to `Home.js` view? We will need a router! (is is already installed with `react-router-dom` package)

Let's go to `App.js` file.

```js
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

We added `BrowsableRouter` with `Switch` and one `Route` to `path="/"` with route to our `Home` component. We can add more components. We will add them in separate directories, because later we will create actions and reducer for each (the Redux part).

Let's add `signup`, `login`, `dashboard` directories in `frontend/src/components`. 

Please add file with `Signup` component in `frontend/src/componenets/signup/Signup.js`:

```js
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

Please add file with `Login` component in `frontend/src/componenets/login/Login.js`:

```js
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


Please add file with `Dashboard` component in `frontend/src/componenets/dashboard/Dashboard.js`:

```js
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

```js
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

OK, time to test it! You can enter in the browser different paths and check if correct components are displayed. For example, please enter [`http://localhost:3000/login`](http://localhost:3000/login) and you should see the login component. 

Maybe you wonder what is the difference between `Route path` and `Route exact path` (used for `Home`). You can try to check it yourself:

- please enter: [`http://localhost:3000/signup/bla/bla/bla`](http://localhost:3000/signup/bla/bla/bla)
- please enter: [`http://localhost:3000/bla/bla/bla`](http://localhost:3000/bla/bla/bla)

The first one will show you `Signup` component, while the second one will show blank window. The first URL will match the `Signup` route. The second one will not have any match.  If you would like to have a default match for any route you can add one more route:

```js
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

