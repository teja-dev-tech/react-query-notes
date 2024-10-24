**What is React Query?**

It is a library for fetching data in a React application.

**Why?**

1. Since React is a UI library, there is no specific pattern for data fetching.
2. We typically use the `useEffect` hook for data fetching and `useState` hook to maintain component state like loading, error state, or the resulting data.
3. If the data is needed throughout the app, then we tend to use state management libraries like Redux.
4. Most of the state management libraries are good for working with client state. Examples: 'theme' for an application / whether a modal is open.
5. State management libraries are not great for working with asynchronous or server state.

**Client state:**

- Persisted in your app memory and accessing or updating it is synchronous.

**Server state:**

- Persisted remotely and requires asynchronous APIs for fetching or updating.
- Has shared ownership.
- Data can be updated by someone else without your knowledge.
- UI data may not be in sync with the remote server/database data.

Challenging when you have to deal with caching, deduplication of multiple requests for the same data, updating stale data in the background, performance optimizations in pagination and lazy-loading, etc.

**Use Query**

In TanStack Query, by default, **inactive queries** (those not being actively used or observed) are garbage collected after **`5 minutes`** . This helps free up memory for unused queries. You can customize the garbage collection behavior using the `cacheTime` option when configuring queries.

In TanStack Query:

- **`isLoading`**: `true` when a query is fetching **for the first time** (no cached data yet).
- **`isFetching`**: `true` when a query is **currently fetching** data, regardless of whether it's the first fetch or a background refetch (cached data may be present).

Query instances via useQuery or useInfiniteQuery by default **consider cached data as stale**.

> To change this behavior, you can configure your queries both globally and per-query using the `staleTime` option. Specifying a longer staleTime means queries will not refetch their data as often
> 

## Polling

`refetchInterval` in TanStack Query is the time (in milliseconds) to automatically refetch the query data at regular intervals. Setting it to a value (e.g., `5000` for 5 seconds) makes the query refetch periodically, while setting it to `0` disables this behavior.

<aside>
💡 If we switch browser tab polling stops, to poll in background `use refetchIntervalInBackground`

</aside>

enabled : 
• Set this to false to disable this query from automatically running.

## Importance of Id in react query

In React Query, each query needs a unique key to differentiate between different queries. If `postId` isn't included as part of the key, React Query may temporarily show cached data from a different query (e.g., showing post 1's data for a split second before fetching the correct data). To prevent this, ensure that `postId` is part of the query key.

### Example:

```
const { data, isLoading } = useQuery(['post', postId], () => fetchPost(postId));

```

Here, `['post', postId]` ensures each `postId` gets its own cached data, avoiding confusion.

## PlaceholderData

In React Query, `placeholderData` can be used to show **previous data** (or any placeholder) while new data is being fetched. It helps provide a smooth user experience by displaying something before the actual data arrives.

### Example (using previous data):

```
const { data, isLoading } = useQuery(
  ['post', postId],
  () => fetchPost(postId),
  {
    placeholderData: () => previousData // using previous data as placeholder
  }
);

```

Here, `placeholderData` will show the previous cached data (if available) until the new data is fetched.

**Infinite query**
```tsx
const { data, fetchNextPage, hasNextPage, isFetchingNextPage } = useInfiniteQuery(
  'posts',
  ({ pageParam = 1 }) => fetchPosts(pageParam), // Fetch function with page param
  {
    getNextPageParam: (lastPage, pages) => {
      return lastPage.hasMore ? pages.length + 1 : undefined; // Determine if there are more pages
    }
  }
);

// Trigger load more
<button onClick={() => fetchNextPage()} disabled={!hasNextPage || isFetchingNextPage}>
  {isFetchingNextPage ? 'Loading more...' : hasNextPage ? 'Load More' : 'No More Pages'}
</button>

```
