# Optimizing React Context Pattern

This technique is used in [Storeon](https://github.com/storeon/storeon) library.

Handling React state using context makes handling states easier. There is a big drawback of using context is that, all child components are rendered whenever any data in context is updated.
This pattern can be used to optimize React state handling without unnecessary rendering.

## Step 1: Create React Context

```js
let {
  useMemo,
  useContext,
  useState,
  useLayoutEffect,
  useEffect,
  createContext,
  createElement,
  forwardRef
} = require('react');

let StoreContext = createContext();
```

## Step 2: Setup useLayoutEffect Hook

```js
let useIsomorphicLayoutEffect = typeof window !== 'undefined' ? useLayoutEffect : useEffect;
```

## Step 3: Setup Custom Hook Handler

```js
let customContext = context => (...keys) => {
  let store = useContext(context);
  if (process.env.NODE_ENV !== 'production' && !store) {
    throw new Error(
      'Could not find store context value. Please ensure the component is wrapped in a store context';
    )
  }

  let hookState = useState({});

  useIsomorphicLayoutEffect(() => {
    return store.on('@changed', (_, changed) => {
      let changesInKeys = keys.some(key => key in changed);
      if (changesInKeys) {
        hookState[1]({});
      }
    })
  }, []);

  return useMemo(() => {
    let state = store.get();
    let data = {};
    keys.forEach(key => {
      data[key] = state[key];
    });
    data.dispatch = store.dispatch;
    return data;
  }, [hookState[0]]);
};

let useContextStore = customContext(StoreContext);
```

## Step 4: Create State Store Handler

```js
let createStore = modules => {
  let events = {}
  let state = {}

  let store = {
    dispatch(event, data) {
      if (event !== '@dispatch') {
        store.dispatch('@dispatch', [event, data, events[event]])
      }

      if (events[event]) {
        let changes
        events[event].forEach(i => {
          let diff = events[event].includes(i) && i(state, data, store)
          if (diff && typeof diff.then !== 'function') {
            state = { ...state, ...diff }
            changes = { ...changes, ...diff }
          }
        })
        if (changes) store.dispatch('@changed', changes)
      }
    },

    get: () => state,

    on(event, cb) {
      ;(events[event] || (events[event] = [])).push(cb)

      return () => {
        events[event] = events[event].filter(i => i !== cb)
      }
    }
  }

  modules.forEach(i => {
    if (i) i(store)
  })
  store.dispatch('@init')

  return store
}

module.exports = { createStore };
```

## Example:

```js
// Initial state, reducers and business logic are packed in independent modules
let count = store => {
  // Initial state
  store.on('@init', () => ({ count: 0 }))
  // Reducers returns only changed part of the state
  store.on('inc', ({ count }) => ({ count: count + 1 }))
}

const store = createStore([count]);

const Counter = () => {
  // Counter will be re-render only on `state.count` changes
  const { dispatch, count } = useContextStore('count')
  return <button onClick={() => dispatch('inc')}>{count}</button>
}

function App() {
  render(
    <StoreContext.Provider value={store}>
      <Counter />
    </StoreContext.Provider>,
    document.body
  )
}
```
