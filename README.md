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
## **React Intersection Observer**

To implement infinite scrolling using the **React Intersection Observer** package, you can follow these steps:

### 1. **Install the Intersection Observer Package**

First, install the `react-intersection-observer` package:

```bash
npm install react-intersection-observer

```

### 2. **Set Up the Infinite Scroll Component**

Here’s an example of how to use the package with `useInfiniteQuery` from React Query:

```jsx
import React from 'react';
import { useInfiniteQuery } from 'react-query';
import { useInView } from 'react-intersection-observer';

const fetchPosts = async ({ pageParam = 1 }) => {
  const response = await fetch(`/api/posts?page=${pageParam}`);
  return response.json();
};

const InfiniteScrollPosts = () => {
  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
  } = useInfiniteQuery('posts', fetchPosts, {
    getNextPageParam: (lastPage, pages) => {
      return lastPage.hasMore ? pages.length + 1 : undefined;
    },
  });

  // Intersection Observer hook
  const { ref: loadMoreRef, inView } = useInView({
    threshold: 1,
    triggerOnce: false, // Set to true if you want to load only once
  });

  React.useEffect(() => {
    if (inView && hasNextPage) {
      fetchNextPage();
    }
  }, [inView, fetchNextPage, hasNextPage]);

  return (
    <div>
      {data?.pages.map((page, i) => (
        <React.Fragment key={i}>
          {page.posts.map((post) => (
            <div key={post.id} className="post">
              {post.title}
            </div>
          ))}
        </React.Fragment>
      ))}

      <div ref={loadMoreRef} style={{ height: '20px' }}>
        {isFetchingNextPage ? 'Loading more...' : hasNextPage ? 'Load more' : 'No more posts'}
      </div>
    </div>
  );
};

export default InfiniteScrollPosts;

```

### 3. **Explanation:**

- The `useInView` hook from `react-intersection-observer` provides a `ref` that you attach to the load-more element.
- When the load-more element comes into view, the `inView` variable becomes `true`, triggering the `fetchNextPage` function.
- The `threshold` option determines how much of the element needs to be visible before the callback is triggered. You can adjust it as needed.

## Mutation

Here’s a simple example of using `useMutation` in React Query for submitting data (like creating a new post):

```tsx
import React from 'react';
import { useMutation, useQueryClient } from 'react-query';

const createPost = async (newPost) => {
  const response = await fetch('/api/posts', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(newPost),
  });
  return response.json();
};

const CreatePost = () => {
  const queryClient = useQueryClient();

  const mutation = useMutation(createPost, {
    onSuccess: (newPost) => {
      // Update the posts cache with the new post
      queryClient.setQueryData('posts', (oldData) => [...oldData, newPost]);
    },
  });

  const handleSubmit = (event) => {
    event.preventDefault();
    const form = event.target;
    const newPost = { title: form.title.value };

    mutation.mutate(newPost); // Call the mutation
    form.reset();
  };

  return (
    <form onSubmit={handleSubmit}>
      <input type="text" name="title" placeholder="Post Title" required />
      <button type="submit">Create Post</button>
      {mutation.isLoading && <p>Adding post...</p>}
      {mutation.isError && <p>Error: {mutation.error.message}</p>}
      {mutation.isSuccess && <p>Post added!</p>}
    </form>
  );
};

export default CreatePost;

```

---

**Optimistic Updates**

As the name indicates, it implies updating the state before performing a mutation under the assumption that nothing can go wrong.

It is typically done to give an impression that your app is blazing fast.

With that said, you do have to cater to scenarios where the mutation can, in fact, error out.

Managing optimistic updates is typically not-so-straightforward, but React Query, on the other hand, does simplify it to a very good extent.

### Example Code:

```jsx
import React from 'react';
import { useMutation, useQueryClient } from 'react-query';

const createPost = async (newPost) => {
  const response = await fetch('/api/posts', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(newPost),
  });
  return response.json();
};

const CreatePost = () => {
  const queryClient = useQueryClient();

  const mutation = useMutation(createPost, {
    // Optimistic update example
    onMutate: async (newPost) => {
      await queryClient.cancelQueries('posts');
      const previousPosts = queryClient.getQueryData('posts');
      queryClient.setQueryData('posts', (old) => [...old, newPost]);
      return { previousPosts };
    },
    onError: (err, newPost, context) => {
      queryClient.setQueryData('posts', context.previousPosts);
    },
    onSettled: () => {
      queryClient.invalidateQueries('posts');
    },
  });

  const handleSubmit = (event) => {
    event.preventDefault();
    const form = event.target;
    const newPost = { title: form.title.value };

    mutation.mutate(newPost); // Call the mutation
    form.reset();
  };

  return (
    <form onSubmit={handleSubmit}>
      <input type="text" name="title" placeholder="Post Title" required />
      <button type="submit">Create Post</button>
      {mutation.isLoading && <p>Adding post...</p>}
      {mutation.isError && <p>Error: {mutation.error.message}</p>}
      {mutation.isSuccess && <p>Post added!</p>}
    </form>
  );
};

export default CreatePost;

```

### Key Points:

- **`useMutation`** is used for creating, updating, or deleting data.
- The `mutation` object provides states like `isLoading`, `isError`, and `isSuccess` to handle UI feedback.
- You can perform optimistic updates and rollback on errors using the `onMutate` and `onError` callbacks.

## Why Cancel Queries

When you perform an optimistic update, you want to prevent any in-flight queries from invalidating your optimistic update. Here's why:

1. Prevent Race Conditions:
If there's an ongoing fetch for the "posts" query, and you add a new post using the mutate function, there's a chance that the ongoing fetch might complete after your optimistic update, overriding the optimistic data with the stale data from the server.
2. Ensure Consistent State:
By cancelling the ongoing queries, you ensure that no stale or incomplete data is accidentally re-applied to the UI after the optimistic update. This keeps the UI in a consistent state.
