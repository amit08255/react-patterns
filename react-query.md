# Handling Data with React Query

React Query library provides hooks for fetching, caching and updating asynchronous data in React. React Query allows you to defeat and overcome the tricky challenges and hurdles of server state and control your app data before it starts to control you.

> npm i react-query

## Query

A query is a declarative dependency on an asynchronous source of data that is tied to a unique key. A query can be used with any Promise based method (including GET and POST methods) to fetch data from a server.

> To subscribe to a query in your components or custom hooks, call the useQuery hook with at least:
> * A unique key for the query
> * A function that returns a promise that: Resolves the data, or Throws an error

```js
import { useQuery } from 'react-query'

function App() {
 const info = useQuery('todos', fetchTodoList)
}
```

> The unique key you provide is used internally for refetching, caching, and sharing your queries throughout your application.

The query results returned by useQuery contains all of the information about the query that you'll need for templating and any other usage of the data.

## States

A query can only be in one of the following states at any given moment:

* ```isLoading``` or ```status === 'loading'``` - The query has no data and is currently fetching
* ```isError``` or ```status === 'error'``` - The query encountered an error
* ```isSuccess``` or ```status === 'success'``` - The query was successful and data is available
* ```isIdle``` or ```status === 'idle'``` - The query is currently disabled (you'll learn more about this in a bit)
* ```error``` - If the query is in an ```isError``` state, the error is available via the error property.
* ```data``` - If the query is in a ```success``` state, the data is available via the data property.
* ```isFetching``` - In any state, if the query is fetching at any time (including background refetching) isFetching will be true.

```js
import { useQuery } from 'react-query'

function Todos() {
 const { isLoading, isError, data, error } = useQuery('todos', fetchTodoList)

 if (isLoading) {
   return <span>Loading...</span>
 }

 if (isError) {
   return <span>Error: {error.message}</span>
 }

 // We can assume by this point that `isSuccess === true`
 return (
   <ul>
     {data.map(todo => (
       <li key={todo.id}>{todo.title}</li>
     ))}
   </ul>
 )
}
```

### Query Keys

Query keys can be as simple as a string, or as complex as an array of many strings and nested objects. As long as the query key is serializable, and unique to the query's data, you can use it!

```js
// A list of todos
useQuery('todos', ...) // queryKey === ['todos']

// Something else, whatever!
useQuery('somethingSpecial', ...) // queryKey === ['somethingSpecial']

// An individual todo
useQuery(['todo', 5], ...) // queryKey === ['todo', 5]

// An individual todo in a "preview" format
useQuery(['todo', 5, { preview: true }], ...) // queryKey === ['todo', 5, { preview: true }]

// A list of todos that are "done"
useQuery(['todos', { type: 'done' }], ...) // queryKey === ['todos', { type: 'done' }]
```

## Query Functions

A query function can be literally any function that returns a promise. The promise that is returned should either resolve the data or throw an error.

```js
useQuery(['todos'], fetchAllTodos)

useQuery(['todos', todoId], () => fetchTodoById(todoId))

useQuery(['todos', todoId], async () => {
  const data = await fetchTodoById(todoId)
  return data
})

useQuery(['todos', todoId], ({ queryKey }) => fetchTodoById(queryKey[1]))
```

### Handling and Throwing Errors

> For React Query to determine a query has errored, the query function must throw. Any error that is thrown in the query function will be persisted on the error state of the query.

```js
const { error } = useQuery(['todos', todoId], async () => {
   if (somethingGoesWrong) {
     throw new Error('Oh no!')
   }

   return data
 })
```
> Explicitly throwing errors when required

```js
useQuery(['todos', todoId], async () => {
   const response = await fetch('/todos/' + todoId)

   if (!response.ok) {
     throw new Error('Network response was not ok')
   }

   return response.json()
 })
```

### Access query key in query function

Query keys are not just for uniquely identifying the data you are fetching, but are also conveniently passed into your query function and while not always necessary, this makes it possible to extract your query functions if needed:

```js
function Todos({ status, page }) {
 const result = useQuery(['todos', { status, page }], fetchTodoList)
}

// Access the key, status and page variables in your query function!
function fetchTodoList({ queryKey }) {
 const [_key, { status, page }] = queryKey
 return new Promise()
}
```

## Dynamic Parallel Queries

If the number of queries you need to execute is changing from render to render, you cannot use manual querying since that would violate the rules of hooks. Instead, React Query provides a useQueries hook, which you can use to dynamically execute as many queries in parallel as you'd like.

> useQueries accepts an array of query options objects and returns an array of query results:

```js
function App({ users }) {
   const userQueries = useQueries(
     users.map(user => {
       return {
         queryKey: ['user', user.id],
         queryFn: () => fetchUserById(user.id),
       }
     })
   )
}
```

## Dependent Queries

> To achieve this, it's as easy as using the enabled option to tell a query when it is ready to run

```js
// Get the user
const { data: user } = useQuery(['user', email], getUserByEmail)

const userId = user?.id

// Then get the user's projects
const { isIdle, data: projects } = useQuery(
 ['projects', userId],
 getProjectsByUser,
 {
   // The query will not execute until the userId exists
   enabled: !!userId,
 }
)



// isIdle will be `true` until `enabled` is true and the query begins to fetch.

// It will then go to the `isLoading` stage and hopefully the `isSuccess` stage :)
```

## Global Loader Indicator for all Queries

> If you would like to show a global loading indicator when any queries are fetching (including in the background), you can use the useIsFetching hook:

```js
import { useIsFetching } from 'react-query'

function GlobalLoadingIndicator() {
 const isFetching = useIsFetching()

 return isFetching ? (
   <div>Queries are fetching in the background...</div>
 ) : null
}
```

## Refetch on Window Focus

> If a user leaves your application and returns to stale data, React Query automatically requests fresh data for you in the background. You can disable this globally or per-query using the refetchOnWindowFocus option:

```js
// Disabling auto refetch globally

const queryClient = new QueryClient({
 defaultOptions: {

   queries: {

     refetchOnWindowFocus: false,

   },

 },

})

function App() {
 return <QueryClientProvider client={queryClient}>...</QueryClientProvider>
}
```

## Query Retries

When a useQuery query fails (the query function throws an error), React Query will automatically retry the query if that query's request has not reached the max number of consecutive retries (defaults to 3) or a function is provided to determine if a retry is allowed.

> You can configure retries both on a global level and an individual query level.
> * Setting retry = false will disable retries.
> * Setting retry = 6 will retry failing requests 6 times before showing the final error thrown by the function.
> * Setting retry = true will infinitely retry failing requests.
> * Setting retry = (failureCount, error) => ... allows for custom logic based on why the request failed.

```js
import { useQuery } from 'react-query'

// Make a specific query retry a certain number of times

const result = useQuery(['todos', 1], fetchTodoListPage, {
 retry: 10, // Will retry failed requests 10 times before displaying an error
})
```

The default retryDelay is set to double (starting at 1000ms) with each attempt, but not exceed 30 seconds:

```js
// Configure for all queries
import { QueryCache, QueryClient, QueryClientProvider } from 'react-query'

const queryClient = new QueryClient({
 defaultOptions: {
   queries: {
     retryDelay: attemptIndex => Math.min(1000 * 2 ** attemptIndex, 30000),
   },
 },
})

function App() {
 return <QueryClientProvider client={queryClient}>...</QueryClientProvider>
}
```

```js
const result = useQuery('todos', fetchTodoList, {
 retryDelay: 1000, // Will always wait 1000ms to retry, regardless of how many retries
})
```

## Query Pagination

```js
const result = useQuery(['projects', page], fetchProjects)
```

> By setting keepPreviousData to true we get a few new things:
> * The data from the last successful fetch available while new data is being requested, even though the query key has changed.
> * When the new data arrives, the previous data is seamlessly swapped to show the new data.
> * ```isPreviousData``` is made available to know what data the query is currently providing you

```js
function Todos() {
 const [page, setPage] = React.useState(0)
 const fetchProjects = (page = 0) => fetch('/api/projects?page=' + page).then((res) => res.json())

 const {
   isLoading,
   isError,
   error,
   data,
   isFetching,
   isPreviousData,
 } = useQuery(['projects', page], () => fetchProjects(page), { keepPreviousData : true })

 return (
   <div>
     {isLoading ? (
       <div>Loading...</div>
     ) : isError ? (
       <div>Error: {error.message}</div>
     ) : (
       <div>
         {data.projects.map(project => (
           <p key={project.id}>{project.name}</p>
         ))}
       </div>
     )}

     <span>Current Page: {page + 1}</span>
     <button
       onClick={() => setPage(old => Math.max(old - 1, 0))}
       disabled={page === 0}
     >
       Previous Page
     </button>{' '}
     <button
       onClick={() => {
         if (!isPreviousData && data.hasMore) {
           setPage(old => old + 1)
         }
       }}

       // Disable the Next Page button until we know a next page is available
       disabled={isPreviousData || !data?.hasMore}
     >
       Next Page
     </button>
     {isFetching ? <span> Loading...</span> : null}{' '}
   </div>
 )
}
```

## Infinite Queries

Rendering lists that can additively "load more" data onto an existing set of data or "infinite scroll" is also a very common UI pattern. React Query supports a useful version of useQuery called useInfiniteQuery for querying these types of lists.

> When using ```useInfiniteQuery```, you'll notice a few things are different:
> * ```data``` is now an object containing infinite query data:
> * ```data.pages``` array containing the fetched pages
> * ```data.pageParams``` array containing the page params used to fetch the pages
> * The ```fetchNextPage``` and ```fetchPreviousPage``` functions are now available
> * A ```hasNextPage``` boolean is now available and is true if getNextPageParam returns a value other than undefined
> * A ```hasPreviousPage``` boolean is now available and is true if getPreviousPageParam returns a value other than undefined
> * The ```isFetchingNextPage``` and ```isFetchingPreviousPage``` booleans are now available to distinguish between a background refresh state and a loading more state

```js
import { useInfiniteQuery } from 'react-query'

function Projects() {
 const fetchProjects = ({ pageParam = 0 }) =>
   fetch('/api/projects?cursor=' + pageParam)

 const {
   data,
   error,
   fetchNextPage,
   hasNextPage,
   isFetching,
   isFetchingNextPage,
   status,
 } = useInfiniteQuery('projects', fetchProjects, {
   getNextPageParam: (lastPage, pages) => lastPage.nextCursor,
 })

 return status === 'loading' ? (
   <p>Loading...</p>
 ) : status === 'error' ? (
   <p>Error: {error.message}</p>
 ) : (
   <>
     {data.pages.map((group, i) => (
       <React.Fragment key={i}>
         {group.projects.map(project => (
           <p key={project.id}>{project.name}</p>
         ))}
       </React.Fragment>
     ))}
     <div>
       <button
         onClick={() => fetchNextPage()}
         disabled={!hasNextPage || isFetchingNextPage}
       >
         {isFetchingNextPage
           ? 'Loading more...'
           : hasNextPage
           ? 'Load More'
           : 'Nothing more to load'}
       </button>
     </div>
     <div>{isFetching && !isFetchingNextPage ? 'Fetching...' : null}</div>
   </>
 )
}
```

> If you only want to actively refetch a subset of all pages, you can pass the refetchPage function to refetch returned from useInfiniteQuery.

```js
const { refetch } = useInfiniteQuery('projects', fetchProjects, {
 getNextPageParam: (lastPage, pages) => lastPage.nextCursor,
})

// only refetch the first page
refetch({ refetchPage: (page, index) => index === 0 })
```

> Skipping pages for fetching data

```js
function Projects() {
 const fetchProjects = ({ pageParam = 0 }) =>
   fetch('/api/projects?cursor=' + pageParam)

 const {
   status,
   data,
   isFetching,
   isFetchingNextPage,
   fetchNextPage,
   hasNextPage,

 } = useInfiniteQuery('projects', fetchProjects, {
   getNextPageParam: (lastPage, pages) => lastPage.nextCursor,
 })

 // Pass your own page param
 const skipToCursor50 = () => fetchNextPage({ pageParam: 50 })
}
```
