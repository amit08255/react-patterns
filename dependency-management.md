# Dependency Management Patterns

Many components (of libraries like React, Vue, Angular) use the functionality of utility libraries.

Let's consider a React component that displays the number of words in the provided text:

```js
import words from 'lodash.words';
function CountWords({ text }: { text: string }): JSX.Element {
  const count = words(text).length;
  return (
    <div className="words-count">{count}</div>
  );
}
```

The component CountWords uses the library lodash.words to count the number of words in the string text.

CountWords component has a dependency on lodash.words library.

The components using dependencies benefit from the code reuse: you simply import the necessary library and use it.

> Designing correctly the dependencies is an important skill to architect Front-end applications. The first step to creating a good design is to identify the stable and volatile dependencies and treat them accordingly.


## Stable dependencies

Let's recall the example component CountWords:

```js
import words from 'lodash.words';
function CountWords({ text }: { text: string }): JSX.Element {
  const count = words(text).length;
  return (
    <div className="words-count">{count}</div>
  );
}
```
The component CountWords is going to use the same library lodash.words no matter the environment: be it on client-side, be it running on the server-side (if you implement Server-Side Rendering), or even when running unit tests.
The signature of words function won't change much in the future.

> Because the dependent component always uses one dependency implementation, and the dependency won't change in the future — such dependency is considered stable.

![image](https://user-images.githubusercontent.com/28493237/147804091-a27689c0-5230-4956-9468-96a551db7179.png)

> You can use them safely and depend directly upon them.


## Volatile dependencies

Consider a Front-end application that supports also Server-Side Rendering.
Example is cookie management. On the client-side, you can access the cookie from document.cookie property, while on the server-side you'd need to read the HTTP request header cookie.

> The cookie management is a volatile dependency because the component chooses the concrete implementation by environment: client-side or server-side.

Generally, the **dependency is volatile if any of the following criteria are met**:

* The dependency requires runtime environment setup for the application (network access, web services, file system)
* The dependency is in development
* The dependency has non-deterministic behavior (random number generator, access of current date, etc).

An example of volatile dependency is, as mentioned, the cookie management library which has different implementations on client and server-side. Another example of volatile dependency is the library to access a database or a fetching library that accesses the network.

> A good rule of thumb to distinguish a volatile dependency is to analyze how easy you can unit test the component that depends on it. If the dependency requires a lot of setup ceremony and mocks to be tested (e.g. a fetching library requires mocking network requests), then most likely it's a volatile.

**Your component should not directly import volatile dependencies.**

![image](https://user-images.githubusercontent.com/28493237/147804216-2c5dca5d-295f-42c7-8f48-2463e41d2efa.png)

* Tight coupling to all dependency implementations.
* Dependency on the environment.
* Unnecessary code.
* Difficult testing.

> The idea consists in applying the Dependency Inversion Principle and decouple volatile dependencies.

First, let's define an interface Cookie that describes what methods a cookie library should implement:

```js
// Cookie.ts
export interface Cookie {
  get(name: string): string | null;
  set(name: string, value: string): void;
}
```

Now let's define the React context that's going to hold a specific implementation of the cookie management library:

```js
// CookieContext.tsx
import { createContext } from 'react';
import { Cookie } from './Cookie';
export const CookieContext = createContext<Cookie>(null);
```

CookieContext injects the dependency into the Page component:

```js
// Page.tsx
import { useContext } from 'react';
import { Cookie }        from './Cookie';
import { CookieContext } from './CookieContext';
import { LoginForm }     from './LoginForm';
export function Page(): JSX.Element {
  const cookie: Cookie = useContext(cookieContext);
  if (cookie.get('loggedIn') === '1') {
    return <div>You are logged in</div>;
  } else {
    return <LoginForm />
  }
}
```

The only thing that Page component knows about is the Cookie interface, and nothing more. The component is decoupled from the implementation details of how cookies are accessed.
Page component doesn't care about what concrete implementation it gets. The only requirement is that the injected dependency to conform to the Cookie interface.

![image](https://user-images.githubusercontent.com/28493237/147804326-0d3c7a57-720f-4bcc-a8af-dae1eef21863.png)

The necessary implementation of the cookie management library is setup by the bootstrap scripts on both client and server sides.

```js
// index.client.tsx
import ReactDOM from 'react-dom';
import { Page }          from './Page';
import { CookieContext } from './CookieContext';
import { cookieClient }  from './libs/cookie-client';
ReactDOM.hydrate(
  <CookieContext.Provider value={cookieClient}>
    <Page />
  </CookieContext.Provider>,
  document.getElementById('root')
);
```

```js
// index.server.tsx
import express from 'express';
import { renderToString } from 'react-dom/server';
import { Page }          from './Page';
import { CookieContext } from './CookieContext';
import { cookieServer }  from './libs/cookie-server';
const app = express();
app.get('/', (req, res) => {
  const content = renderToString(
    <CookieContext.Provider value={cookieServer}>
      <Page />
    </CookieContext.Provider>
  );
  res.send(`
    <html>
      <head><script src="./bundle.js"></script></head>
      <body>
        <div id="root">
          ${content}
        </div>
      </body>
    </html>
  `);
})
app.listen(env.PORT ?? 3000);
```

The benefits of good design of volatile dependencies:
* Loose coupling
* Free of implementation details and environment
* Dependency upon stable abstraction
* Easy testing

```js
// Page.tsx
import { Cookie }        from './Cookie';
import { LoginForm }     from './LoginForm';
interface PageProps {
  cookie: Cookie;
}
export function Page({ cookie }: PageProps): JSX.Element {
  if (cookie.get('loggedIn') === '1') {
    return <div>You are logged in</div>;
  } else {
    return <LoginForm />
  }
}
```
