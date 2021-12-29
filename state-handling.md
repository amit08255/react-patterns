# React State Handling Pattern

The state is information hidden inside a component. The component can modify its state, without parents knowing about it.

# useState Hook

React.useState is pretty straightforward to use. A value, a setter function, an initial state. The first argument of useState(initialState) is the initial state.

> When the hook useState(initialState) is invoked, it returns an array. The first item of this array is the state value and the second item is a function that updates the state (which is called the "state dispatch function")!

```js
import React, { useState } from 'react';
function Bulbs() {
  const [on, setOn] = useState(false);
  const lightOn = () => setOn(true);
  const lightOff = () => setOn(false);
  return (
    <>
      <div className={on ? 'bulb-on' : 'bulb-off'} />
      <button onClick={lightOn}>On</button>
      <button onClick={lightOff}>Off</button>
    </>
  );
}
```

To update the component's state invoke the updater function setState(newState) with the new state. The component re-renders and state receives the new value newState.

> When you call the state dispatch function, you pass the new value for the state and that triggers a re-render of the component which leads to useState getting called again to retrieve the new state value and the dispatch function again.


### Lazy Initialization

So if you pass a function to useState, React will only call the function when it needs the initial value (which is when the component is initially rendered).

This is called "lazy initialization." It's a performance optimization. You shouldn't have to use it a whole lot, but it can be useful in some situations, so it's good to know that it's a feature that exists and you can use it when needed. I would say I use this only 2% of the time. It's not really a feature I use often.

```js
const getInitialState = () => Number(window.localStorage.getItem('count'))
const [count, setCount] = React.useState(getInitialState)
```


### The functional updater

Instead of passing a new value to the setter that we get from useState, we can also pass a function to it. React will call that function and gives us the previousValue, so that we can calculate a new result depending on it:

```js
const [count, setCount] = React.useState(0)

// 🚨 depends on the current count value to calculate the next value
<button onClick={() => setCount(count + 1)}>Increment</button>

// ✅ uses previousCount to calculate next value
<button onClick={() => setCount(previousCount => previousCount + 1)}>Increment</button>
```
> The useState updater only schedules an update. It basically tells React:
> * Please set this value to the new value, somewhen.

Any time I need to compute new state based on previous state, I use a function update. **This is safer way for updating states in components.**

```js
function App() {
  const [count, setCount] = React.useState(0)

  return (
    <button
      onClick={() => {
        setCount((previousCount) => previousCount + 1)
        setCount((previousCount) => previousCount + 1)
      }}
    >
      ✅ Increment by 2, count is: {count}
    </button>
  )
}

render(<App />)
```

## useReducer hook

**useReducer**: Allows you to combine multiple states in one place instead of use separate useState for every state.
Accepts a reducer of type ```(state, action) => newState```, and returns the current state paired with a dispatch method. useReducer is usually ```preferable to useState when you have complex state logic``` that involves multiple sub-values ```or when the next state depends on the previous one```.

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

> Safe set state with check if component is mounted to prevent warning showing cannot set state to unmounted component

```js
import { useReducer, useEffect, useContext, useRef } from 'react';

function Query({ query, variables, children, normalize = data => data }) {
  const client = useContext(GitHub.Context);

  const [state, setState] = useReducer(
    (state, newState) => ({ ...state, ...newState }),
    { loaded: false, fetching: false, data: null, error: null }
  );
  
  const mountedRef = useRef(false);
  
  useEffect(() => {
    mountedRef.current = true;
    return () => (mountedRef.current = false);
  }, []);
  
  const safeSetState = (...args) => mountedRef.current && setState(...args);
  
  useEffect(
    () => {
      setState({ fetching: true });

      client.request(query, variables).then(res => safeSetState({
        data: normalize(res),
        error: null,
        loaded: true,
        fetching: false,
      })).catch(error => safeSetState({
        error,
        data: null,
        loaded: false,
        fetching: false,
      }))
    }, [query, variables]);
    
    return children(state);
}
```
