# Sharing Code with Hooks Pattern

React hooks are great for sharing code between multiple component. We will use functional components for this pattern.

## React Hooks Summary

### useContext hook

* **useContext**: Allows you to use React context in component. Accepts a context object (the value returned from React.createContext) and returns the current context value for that context. The current context value is determined by the value prop of the nearest ```<MyContext.Provider>``` above the calling component in the tree. When the nearest ```<MyContext.Provider>``` above the component updates, this Hook will trigger a rerender with the latest context value passed to that MyContext provider.

```js
import { useContext } from 'react';

const themes = {
  light: {
    foreground: "#000000",
    background: "#eeeeee"
  },
  dark: {
    foreground: "#ffffff",
    background: "#222222"
  }
};

const ThemeContext = React.createContext(themes.light);

function App() {
  return (
    <ThemeContext.Provider value={themes.dark}>
      <Toolbar />
    </ThemeContext.Provider>
  );
}

function Toolbar(props) {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

function ThemedButton() {
  const theme = useContext(ThemeContext);
  return (
    <button style={{ background: theme.background, color: theme.foreground }}>
      I am styled by theme context!
    </button>
  );
}
```

### useRef hook

* **useRef**: useRef returns a mutable ref object whose .current property is initialized to the passed argument (initialValue). The returned ```object will persist for the full lifetime of the component``` which means useRef will give you the same ref object on every render.

```js
import {useRef} from 'react';

function TextInputWithFocusButton() {
  const inputEl = useRef(null);
  const onButtonClick = () => {
    // `current` points to the mounted text input element
    inputEl.current.focus();
  };
  return (
    <>
      <input ref={inputEl} type="text" />
      <button onClick={onButtonClick}>Focus the input</button>
    </>
  );
}
```


### useEffect hook

* **useEffect**: The function passed to useEffect will run after the render is committed to the screen. By default, effects run after every completed render, but you can choose to fire them only when certain values have changed by passing second parameter which is array of values to check and render when any of them changes.
When the second parameter is empty array, the callback is called only once when component is mounted. The default behavior for effects is to fire the effect after every completed render.
If a function is returned from useEffect, that function will be executed when the component is unmounted from screen.  The function passed to useEffect fires after layout and paint, during a deferred event.

> useEffect with callback to run when component unmounts

```js
import { useEffect } from 'react';

function Query(props) {
  useEffect(() => {
    const subscription = props.source.subscribe();
    return () => {
      // Clean up the subscription
      subscription.unsubscribe();
    };
  });
  
  return <div>Hello</div>;
}
```

> useEffect with empty dependency to call only once when mounted

```js
import { useEffect } from 'react';

function Query() {
  useEffect(() => {
    console.log("component mounted");
  }, []);
  
  return <div>Hello</div>;
}
```

> useEffect to execute every time a value changes

```js
import { useEffect } from 'react';

function Query(props) {
  useEffect(() => {
    console.log("component source value mounted");
  }, [props.source]);
  
  return <div>Hello</div>;
}
```

**Note:** Values under useEffect dependencies are not compared using shallow comparison instead it is compared using direct comparison like: ```val1 === val2```. When an object will contain same value but different instance is passed, then it will be considered a new value.

> Custom value comparison technique in useEffect

```js
import { useEffect, useRef } from 'react';
import isEqual from 'lodash/isEqual';

function Query({ query, variables }) {
  useEffect(() => {
    if(isEqual(previousInput.current, [query, variables]) {
      return;
    }

    console.log("component source value mounted");
  });
  
  const previousInput = useRef();
  
  useEffect(() => {
    previousInput.current = [query, variables];
  });
  
  return <div>Hello</div>;
}
```


### useReducer hook

* **useReducer**: Allows you to combine multiple states in one place instead of use separate useState for every state.
Accepts a reducer of type ```(state, action) => newState```, and returns the current state paired with a dispatch method. useReducer is usually ```preferable to useState when you have complex state logic``` that involves multiple sub-values or when the next state depends on the previous one.

```js
const initialState = {count: 0};

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return {count: state.count + 1};
    case 'decrement':
      return {count: state.count - 1};
    default:
      throw new Error();
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({type: 'decrement'})}>-</button>
      <button onClick={() => dispatch({type: 'increment'})}>+</button>
    </>
  );
}
```

> It is very simple to implement useState hook using useReducer hook.

```js
// simulating setState with useReducer

import { useReducer, useEffect, useContext } from 'react';

function Query({ query, variables, children, normalize = data => data }) {
  const client = useContext(GitHub.Context);

  const [state, setState] = useReducer(
    (state, newState) => ({ ...state, ...newState }),
    { loaded: false, fetching: false, data: null, error: null }
  );
  
  useEffect(
    () => {
      setState({ fetching: true });

      client.request(query, variables).then(res => setState({
        data: normalize(res),
        error: null,
        loaded: true,
        fetching: false,
      })).catch(error => setState({
        error,
        data: null,
        loaded: false,
        fetching: false,
      }))
    }, [query, variables]);
    
    return children(state);
}
```

