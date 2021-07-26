# How do you make sure that user remains authenticated on page refresh while using Context API State Management?

The basic idea to persist any data in react app on page refresh is to store the data outside the app and update the states based on the stored data. The store can be either a server side database (sql or no-sql), or localStorage, sessionStorage, cookies on the client-side (browser).

- When using context API, we can create a context provider component namely `ContextProvider` and then wrap the App component in `index.js` with that provider component.

- Then, in App component, we can initiate a `useEffect` hook, there we can take the **auth token** and the **encrypted password** that has already been saved in the browser's storage. That saving process usually happens when a user signs up to a website.

- Then, we can write a function in the ContextProvider component `authUser` that makes a http post request to the server by putting the token and password as post request data.

- And then, in App component, we can call that function inside `useEffect` hook. That will send the token and password to the server. After comparing with the data stored in the server, the server will send a response (preferably a boolean one).

- Because we are making the API call with `useEffect`, every time the App component renders, it will invoke the `useEffect -> authUser` function. And login status will persist.

- After that, we can save the comparison value as state `loginStatus` using useState hook inside `ContextProvider` and update the state value inside the http post request chain we declared in `ContextProvider -> authUser`.

- After accomplishing the above steps, the entire React app will be able to get access to the authentication value `loginStatus` once we import it by using `useContext` hook inside all of our desired components that require state persistance. From there, we can use the data as intended.

Another thing to mention here is that, in most cases, it is not a good idea to save the auth token in session storage of a browser. Because if users close the tab, the session will be cleared. In that case, users have to log in every time they close the tab. So, unless that is what the actually want to achieve, it is better to keep the token in local storage or in cookies with an expiration time.

Also, it is not usually a good idea to solely rely on browser storages without running them through the server-side user data. Because, client-side data can be manipulated easily which can cause security issues.

## Code for the above steps:

### Context.js

```js
import React, { useState, useEffect } from "react";
import axios from "axios"; //run "npm install axios" if not already installed

const Context = React.createContext(null);

const ContextProvider = ({ children }) => {
  const [loginStatus, setLoginStatus] = useState(false);

  // function to make API call
  const authUser = async = (data, url) => {
    try {
      const response = await axios.post(url, authData);

      setLoginStatus(response.data) // can vary based on response is coming as what form of data

    } catch((error) => {
      console.log(error.message)
    })
  }

return (
  <Context.Provider value={{authUser, loginStatus, setLoginStatus }}>
    {children}
  </Context.Provider>);
};

export {Context, ContextProvider};
```

### index.js

```js
import React from "react";
import ReactDOM from "react-dom";
import App from "./App";
import { ContextProvider } from "Where we have kept the Context component";

ReactDOM.render(
  <React.StrictMode>
    <ContextProvider>
      <App />
    </ContextProvider>
  </React.StrictMode>,
  document.getElementById("root")
);
```

### App.js

```js
import React, { useContext, useEffect } from "react";
import { Context } from "Where we have kept the Context component";
import LoginForm from "where it is kept";
import Dashboard from "where it is kept";

const App = () => {
  const { authUser, loginStatus, setLoginStatus } = useContext(Context);

  useEffect(() => {
    const authData = {
      token: localStorage.getItem("token"),
      password: localStorage.getItem("password"),
    };
    const apiUrl = "placeholder/for/api.url";

    // API call
    authUser(authData, apiUrl);
  }, []);

  return (
    <div>
      {!loginStatus && <LoginForm />}
      {loginStatus && <Dashboard />}
    </div>
  );
};

export default App;
```
