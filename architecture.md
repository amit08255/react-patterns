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
* React hooks
* React suspense
* Dependency inversion



