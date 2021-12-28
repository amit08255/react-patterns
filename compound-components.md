# Compound Components

## Introduction

Think of compound components like the \<select\> and \<option\> elements in HTML. Apart they don't do too much, but together they allow you to create the complete experience. The way they do this is by sharing implicit state between the components. Compound components allow you to create and use components which share this state implicitly.

> The idea is that you have two or more components that work together to accomplish a useful task. Typically one component is the parent, and the other is the child. The objective is to provide a more expressive and flexible API.

So the compound components API gives you a nice way to express relationships between components.

> Another important aspect of this is the concept of "implicit state."

The `<select>` element implicitly stores state about the selected option and shares that with it's children so they know how to render themselves based on that state. But that state sharing is implicit because there's nothing in our HTML code that can even access the state (and it doesn't need to anyway).

```js
function App() {
  return (
    <Toggle onToggle={on => console.log(on)}>
      <Toggle.On>The button is on</Toggle.On>
      <Toggle.Off>The button is off</Toggle.Off>
      <Toggle.Button />
    </Toggle>
  )
}
```

The actual full implementation of compound components with context and hooks:

```js
import * as React from 'react'
// this switch implements a checkbox input and is not relevant for this example
import {Switch} from '../switch'

const ToggleContext = React.createContext()

function useEffectAfterMount(cb, dependencies) {
  const justMounted = React.useRef(true)
  React.useEffect(() => {
    if (!justMounted.current) {
      return cb()
    }
    justMounted.current = false
  }, dependencies)
}

function Toggle(props) {
  const [on, setOn] = React.useState(false)
  const toggle = React.useCallback(() => setOn(oldOn => !oldOn), [])
  useEffectAfterMount(() => {
    props.onToggle(on)
  }, [on])
  const value = React.useMemo(() => ({on, toggle}), [on])
  return (
    <ToggleContext.Provider value={value}>
      {props.children}
    </ToggleContext.Provider>
  )
}

function useToggleContext() {
  const context = React.useContext(ToggleContext)
  if (!context) {
    throw new Error(
      `Toggle compound components cannot be rendered outside the Toggle component`,
    )
  }
  return context
}

function On({children}) {
  const {on} = useToggleContext()
  return on ? children : null
}

function Off({children}) {
  const {on} = useToggleContext()
  return on ? null : children
}

function Button(props) {
  const {on, toggle} = useToggleContext()
  return <Switch on={on} onClick={toggle} {...props} />
}

// for convenience, but totally not required...
Toggle.On = On
Toggle.Off = Off
Toggle.Button = Button
```



So the way this works is we create a context with React where we store the state and a mechanism for updating the state. Then the <Toggle> component is responsible for providing that context value to the rest of the react tree
