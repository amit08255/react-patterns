# Sharing Code with Hooks Pattern

React hooks are great for sharing code between multiple component. We will use functional components for this pattern.

## React Hooks Summary

### useContext hook

* **useContext**: Allows you to use React context in component.

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
