# Headless Components Pattern

The headless component pattern in React is one of the most powerful React pattern. Here the component is split into two parts, UI component and hook. The UI component is only responsible for handling the user interface. The hook is responsible for handling everything related to logic for the component. This pattern allows to create resilient, futureproof, highly sharable component codes.

**Example:**

```jsx
import * as React from 'react';
import TabButton from 'components/TabButton';
import useFinanceTabs from 'hooks/useFinanceTabs';

const FinanceTabs = () => {
  const { getTabList, onTabClick } = useFinanceTabs();

  return (
    <div>
      {
        getTabList().map((item) => <TabButton key={item.id} isActive={item.isActive} onClick={onTabClick}>{item.label}</TabClick>)
      }
    </div>
  );
};

export default FinanceTabs;
```

#### Higher-Order Headless Component

This pattern will allow to extend the pattern further using currying. The first parameter will accept a hook function as parameter. This will allow dynamically using custom logic with the UI component.

**Example 2:**

```jsx
// headless component
import * as React from 'react';
import TabButton from 'components/TabButton';

const FinanceTabs = (useFinanceTabs) => () => {
  const { getTabList, onTabClick } = useFinanceTabs();

  return (
    <div>
      {
        getTabList().map((item) => <TabButton key={item.id} isActive={item.isActive} onClick={onTabClick}>{item.label}</TabClick>)
      }
    </div>
  );
};

export default FinanceTabs;
```

```jsx
// using headless component
import * as React from 'react';
import FinanceTabs from 'components/FinanceTabs';
import useBookedFinanceTabs from 'hooks/useBookedFinanceTabs';

const Tabs = FinanceTabs(useBookedFinanceTabs); // Component function

const BookedTabs = () => {
  return (
    <Tabs />
  );
};
```
