# High Order Components Pattern

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

## Manipulating Props

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

## Accessing the instance via Refs

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

## State abstraction

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

## Wrapping the WrappedComponent with other elements

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

## Render Hijacking

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
