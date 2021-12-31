# React Architecture Patterns

## Orthogonal Components

If A and B are orthogonal, then changing A does not change B (and vice-versa). That's the concept of orthogonality.

> In a radio device, the volume and station selection controls are orthogonal. The volume control changes only the sound volume. The station selection control changes only the received radio station.

When you try to add changes to tightly coupled components: you're forced to catch the side-effects of your changes.

> Two or more components are orthogonal if a change in one component does not affect other components.

For example, a component that displays a list of employees should be orthogonal to the logic that fetches the employees.

A good React application design would make orthogonal:
* The UI elements (the presentational components)
* Fetch details (fetch library, REST or GraphQL)
* Global state management (Redux)
* Persistence logic (local storage, cookies)

> Make your components implement one task, be isolated, self-contained and encapsulated. This will make your components orthogonal, and any change you make is going to be isolated and focused on just one component.

**Example: Non-orthogonal component**

```js
import React, { useState } from 'react';
import axios from 'axios';
import EmployeesList from './EmployeesList';
function EmployeesPage() {
  const [isFetching, setFetching] = useState(false);
  const [employees, setEmployees] = useState([]);
  useEffect(function fetch() {
    (async function() {
      setFetching(true);
      const response = await axios.get("/employees");
      setEmployees(response.data);
      setFetching(false);
    })();
  }, []);
  
  if (isFetching) {
    return <div>Fetching employees....</div>;
  }
  return <EmployeesList employees={employees} />;
}
```

*Let's isolate the fetch logic details from the component.* A good way to do this is to use the new Suspense feature of React.

**Example: Orthogonal component**

```js
import React, { Suspense } from "react";
import EmployeesList from "./EmployeesList";
function EmployeesPage({ resource }) {
  return (
    <Suspense fallback={<h1>Fetching employees....</h1>}>
      <EmployeesFetch resource={resource} />
    </Suspense>
  );
}
function EmployeesFetch({ resource }) {
  const employees = resource.employees.read();
  return <EmployeesList employees={employees} />;
}
```

> While isolating changes into separate components is what orthogonality is all about, there could be components that can change for different reasons.

**Ways of making components orthogonal:**
* React hooks or built-in hooks
* React suspense
* Dependency inversion


## Single responsibility

A component has a single responsibility when it has one reason to change.

> A component has one reason to change when it implements one responsibility, or simpler when it does one thing.

A responsibility is either to render a list of items, or to show a date picker, or to make an HTTP request, or to draw a chart, or to lazy load an image, etc.

> Your component should pick only one responsibility and implement it. Ask yourself: do I have to split the component into smaller pieces?

**Example:** Imagine a component that makes an HTTP request to a specialized server to get the current weather. When data is successfully fetched, the same component uses the response to display the weather. The weather component has 2 reasons to change: Fetch logic, Weather visualization. The solution is to divide ```<Weather>``` in 2 components: each having one responsibility. Let's name the chunks ```<WeatherFetch>``` and ```<WeatherInfo>```.

### High Order Components

> HOC favors single responsibility principle: Higher order component is a function that takes one component and returns a new component.

A common usage of HOC is to provide the wrapped component with additional props or modify existing prop values. This technique is called ```props proxy```.

**Example: High Order Component**

```js
function withNewFunctionality(WrappedComponent) {
  return class NewFunctionality extends Component {
    render() {
      const newProp = 'Value';
      const propsProxy = {
         ...this.props,
         // Alter existing prop:
         ownProp: this.props.ownProp + ' was modified',
         // Add new prop:
         newProp
      };
      return <WrappedComponent {...propsProxy} />;
    }
  }
}
const MyNewComponent = withNewFunctionality(MyComponent);
```
> What can be done with Props Proxy?
> * Manipulating props
> * Accessing the instance via Refs
> * Accessing the instance via Refs
> * Wrapping the WrappedComponent with other elements

#### Manipulating Props

You can read, add, edit and remove the props that are being passed to the WrappedComponent.

```js
function ppHOC(WrappedComponent) {
  return class PP extends React.Component {
    render() {
      const newProps = {
        user: currentLoggedInUser
      }
      return <WrappedComponent {...this.props} {...newProps}/>
    }
  }
}
```

#### Accessing the instance via Refs

You can access this (the instance of the WrappedComponent) with a ref, but you will need a full initial normal render process of the WrappedComponent for the ref to be calculated, that means that you need to return the WrappedComponent element from the HOC render method, let React do it’s reconciliation process and just then you will have a ref to the WrappedComponent instance.

```js
function refsHOC(WrappedComponent) {
  return class RefsHOC extends React.Component {
    proc(wrappedComponentInstance) {
      wrappedComponentInstance.method()
    }
    
    render() {
      const props = Object.assign({}, this.props, {ref: this.proc.bind(this)})
      return <WrappedComponent {...props}/>
    }
  }
}
```

When the WrappedComponent is rendered the ref callback will be executed, and then you will have a reference to the WrappedComponent’s instance. This can be used for reading/adding instance props and to call instance methods.

#### State abstraction

You can abstract state by providing props and callbacks to the WrappedComponent, very similar to how smart components will deal with dumb components.

```js
function ppHOC(WrappedComponent) {
  return class PP extends React.Component {
    constructor(props) {
      super(props)
      this.state = {
        name: ''
      }
      
      this.onNameChange = this.onNameChange.bind(this)
    }
    onNameChange(event) {
      this.setState({
        name: event.target.value
      })
    }
    render() {
      const newProps = {
        name: {
          value: this.state.name,
          onChange: this.onNameChange
        }
      }
      return <WrappedComponent {...this.props} {...newProps}/>
    }
  }
}
```

#### Wrapping the WrappedComponent with other elements

You can wrap the WrappedComponent with other components and elements for styling, layout or other purposes.

```js
function ppHOC(WrappedComponent) {
  return class PP extends React.Component {
    render() {
      return (
        <div style={{display: 'block'}}>
          <WrappedComponent {...this.props}/>
        </div>
      )
    }
  }
}
```

#### Render Hijacking

You can also hook into render mechanism by altering elements that wrapped component renders. This HOC technique is named render highjacking.

```js
function withModifiedChildren(WrappedComponent) {
  return class ModifiedChildren extends WrappedComponent {
    render() {
      const rootElement = super.render();
      const newChildren = [
        ...rootElement.props.children, 
        // Insert a new child:
        <div>New child</div>
      ];
      return React.cloneElement(
        rootElement, 
        rootElement.props, 
        newChildren
      );
    }
  }
}
const MyNewComponent = withModifiedChildren(MyComponent);
```

## Encapsulation

An encapsulated component provides props to control its behavior while not exposing its internal structure.

> A well encapsulated component hides its internal structure and provides a set of props to control its behavior.

Units that precisely hide their internal structure are less dependent on each other. Lowering the dependency degree brings the benefits of loose coupling.

Component's instance and state object are implementation details encapsulated inside the component.

> The solution is to design a convenient communication interface that respects loose coupling and strong encapsulation. This can be done using component props and hooks (or custom hooks).

## Composition

A composable component is created from the composition of smaller specialized components.

> Composition is a way to combine components to create a bigger (composed) component.

```js
const app = (
  <Application>
    <Header />
    <Sidebar>
      <Menu />
    </Sidebar>
    <Content>
      <Article />
    </Content>
    <Footer />
  </Application>
);
```

> Single responsibility principle describes how to split requirements into components, encapsulation describes how to organize these components, and composition describes how to glue the whole system back.

This divide and conquer approach helps an authority component conform to single responsibility principle.
Components using composition can reuse common logic. This is the benefit of reusability.

```js
const instance1 = (
  <Composed1>
    /* Specific to Composed1 code... */
    /* Common code... */
  </Composed1>
);
const instance2 = (
  <Composed2>
    /* Common code... */
    /* Specific to Composed2 code... */
  </Composed2>
);
```

User interfaces are composable hierarchical structures. Thus composition of components is an efficient way to construct user interfaces.

## Reusable

A reusable component is written once but used multiple times.

> Components that have only one responsibility are the easiest to reuse.

Correct encapsulation creates a component that doesn't stuck with dependencies. Hidden internal structure and focused props enable the component to fit nicely in multiple places where it's about to be reused.

## Pure

A pure component always renders same elements for same prop values.

> An almost-pure component always renders same elements for same prop values, and can produce a side effect.

```js
function Message({ text }) {
  return <div className="message">{text}</div>;
}

<Message text="Hello World!" /> 
```

## Testable

A testable component is easy to test.

A component that is untestable or hard to test is most likely badly designed.

> A component is hard to test because it has a lot of props, dependencies, requires mockups and access to global variables: that's the sign of a bad design.

When the component has weak architectural design, it becomes untestable. When the component is untestable, you simply skip writing unit tests: as result it remains untested.
