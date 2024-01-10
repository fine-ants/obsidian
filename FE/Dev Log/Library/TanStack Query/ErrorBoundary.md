# TanStack Query and ErrorBoundary

## ErrorBoundary
- `componentDidCatch(error, info)` api를 통해 render 과정에서 일어나는 에러를 잡을 수 있는데, 이는 현재 Class component에만 가능하다.
- ErrorBoundary class component을 만들거나 `react-error-boundary` library를 사용.
	- https://react.dev/reference/react/Component#componentdidcatch-caveats
	- https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary
	- https://react.dev/reference/react/Component#componentdidcatch

## ErrorBoundary with TanStack Query

	
- `useSuspenseQuery` relies on the internal cache for suspense functionality, and the cache operates independently of the `gcTime` configuration. Therefore, error responses are mandatorily cached despite manually setting `gcTime: 0`.
- When using **suspense** or **throwOnError** in your queries, you need a way to let queries know that you want to try again when re-rendering after some error occurred. With the `QueryErrorResetBoundary` component you can reset any query errors within the boundaries of the component.
- [ ] https://tanstack.com/query/v5/docs/react/reference/QueryErrorResetBoundary