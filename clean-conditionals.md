# Writing Clean Coditionals In React

## Introduction

Conditional rendering allows you to build dynamic UI components. This simple pattern will allow you to write cleaner React code with simple conditional pattern.

## Traditional Conditional Examples

### Using &&

Used quite often the "&&" is easy to throw in for some quick conditional logic. The "&&" is only good to use sparingly and only when there is one condition.

**Example 1:**

```tsx
const Session = ({ isLoggedIn }) => {
  return (
    <>
      {isLoggedIn && <Login />}
      {!isLoggedIn && <SignOut />}
    </>
  );
};
```

**Example 2:**

```tsx
const Session = ({ isLoggedIn, isUnicorn }) => {
  <>
    {isLoggedIn && !isUnicorn && <Login />}
    {!isLoggedIn && isUnicorn && <isUnicorn />}
    {!isLoggedIn && !isUnicorn && <SignOut />}
  </>;
};
```

When using "&&" to check the length of a data set and then mapping over it will render "0" when that list is empty.
The code below will render "0" when data is empty.

**Example 3:**

```tsx
const RenderData = ({ data }) =>  data.length && data.map(...); 
```

### Using If/Else Statements

Given this simple scenario, a guard clause works here and is a bit more readable than the "&&".

**Example 1:**

```tsx
const Session = ({ isLoggedIn }) => {
  if (isLoggedIn) {
    return <SignOut />
  }

  return <Login />
};
```

Less awful, but this is going to make things tricky if you ever wanted to wrap each of the components being returned with another component.

**Example 2:**

```tsx
const Session = ({ isLoggedIn, isUnicorn }) => {
  if (isLoggedIn) {
    return <SignOut />;
  } else if (isUnicorn) {
    return <UnicornLogin />;
  };

  return <Login />;
};
```

### Using Ternary Operator

This also is easier to understand; being able to one line this is pretty nice.

**Example 1:**

```tsx
const Session = ({ isLoggedIn }) => isLoggedIn ? <SignOut /> : <Login />;
```

Ternaries are nice if you have one condition. However, it is too easy to not refactor and quickly add another check and then another.

This is a simple example, you can imagine what this would look like if each component we rendered were 5-10 or more lines long.

**Example 2:**

```tsx
const RenderData = ({ data }) => {
  return data.length === 0 ? null
    : data.length === 1
    ? <SingleItem data={data} />
    : data.length === 2
    ? <DoubleItem data={data} />
    : <MultiItem data={data} />
}
```

## Writing Clean Conditionals

### Conditional Operator Component

I think it is easier to read JSX if your brain only has to parse and not conditional statements plus . So, how can we write conditional operators as XML?

Let's consider creating a component called "RenderIf" and takes a boolean property of "isTrue" and renders its children.

**Conditional Component:**

```tsx
export const RenderIf = ({ children, isTrue }) => isTrue ? children : null;

RenderIf.propTypes = {
  children: node.isRequired,
  isTrue: bool.isRequired,
};
```

Re-writing our example with the "RenderIf" component, I'm mostly looking at XML. However, there is still some boolean logic that could be cleaned up.

**Example 1:**

```tsx
const RenderData = ({ data }) => {
  return (
    <>
      <RenderIf isTrue={data.length === 1}>
        <SingleItem data={data} />
      </RenderIf>
      <RenderIf isTrue={data.length === 2}>
        <DoubleItem data={data} />
      </RenderIf>
      <RenderIf isTrue={data.length > 2}>
        <MultiItem data={data} />
      </RenderIf>
    </>
  );
}
```

We can clean up the boolean logic by wrapping our "RenderIf" component.

**Example 2:**

```tsx
const IfSingleItem = ({ children, data }) => <RenderIf isTrue={data.length === 1}>{children}</RenderIf>
const IfDoubleItem = ({ children, data }) => <RenderIf isTrue={data.length === 2}>{children}</RenderIf>
const IfMultiItem = ({ children, data }) => <RenderIf isTrue={data.length > 3}>{children}</RenderIf>

const RenderData = ({ data }) => {
  return (
    <>
      <IfSingleItem data={data}>
        <SingleItem data={data} />
      </IfSingleItem>
      <IfDoubleItem data={data}>
        <DoubleItem data={data} />
      </IfDoubleItem>
      <IfMultiItem data={data}>
        <MultiItem data={data} />
      </IfMultiItem>
    </>
  );
}
```

### Conditional Operator Component To Limit Number Of True Conditions

It works similar to if-else and switch case. If first condition gets true in the children tree, then only that one will be executed.
An optional props `limit` will allow to limit the number of true conditions to render at a time. By default limit is 1.
There is another optional props `isTrue` in ReactWhen which allows you to set condition for entire parent conditional component.

```tsx
import * as React from 'react';

type WhenProps = {
    children: React.ReactNode,
    isTrue?: boolean,
    limit?: number,
};

const RenderWhen = ({ limit, isTrue, children }:WhenProps) => {
    const list = [];

    if (isTrue !== true) {
        return null;
    }

    React.Children.map(children, (child:any) => {
        const { isTrue: isChildTrue } = child?.props || {};

        if (isChildTrue === true && list.length < limit) {
            list.push(child);
        }
    });

    return list;
};

RenderWhen.defaultProps = {
    limit: 1,
    isTrue: true,
};

RenderWhen.If = ({ children, isTrue }) => children;

export default RenderWhen;
```

**Example:**

```tsx
import React from 'react';

export function App(props) {
  const x = 2;

  return (
    <RenderWhen>
      <RenderWhen.If isTrue={x === 2}>
        Hello 2
      </RenderWhen.If>
      <RenderWhen.If isTrue={x === 2}>
        Bye 2
      </RenderWhen.If>
    </RenderWhen>
  );
}
```
