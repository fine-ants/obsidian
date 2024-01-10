# TanStack Query and ErrorBoundary

## ErrorBoundary
- `componentDidCatch(error, info)` api를 통해 render 과정에서 일어나는 에러를 잡을 수 있는데, 이는 현재 Class component에만 가능하다.
- ErrorBoundary class component을 만들거나 `react-error-boundary` library를 사용.
### Reference
- https://react.dev/reference/react/Component#componentdidcatch
- https://react.dev/reference/react/Component#componentdidcatch-caveats
- https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary

## ErrorBoundary with TanStack Query
- 보통의 경우 아래 코드처럼 구조를 ErrorBoundary와 Suspense를 사용하게 된다.
```tsx
<ErrorBoundary>
	<Suspense>
		<MyComponent />
	</Suspense>
</ErrorBoundary>
```
- TanStack Query에서 제공하는 `useSuspenseQuery`를 사용하여 React의 `Suspense`와 함께 TanStack Query를 사용할 수 있게 된다.
- 이 `useSuspenseQuery`는 React의 `Suspense`을 지원하기 위해 내부 cache을 사용하고 이 cache는 `gcTime` 설정과 별개로 운영인된다. 그러므로 `gcTime`을 `0`으로 설정해도 error 응답 마저 필연적으로 cache가 된다.
- 그렇다면, error 발생 후 재시도를 가능하게 하려면 해당 query에 


- When using **suspense** or **throwOnError** in your queries, you need a way to let queries know that you want to try again when re-rendering after some error occurred. With the `QueryErrorResetBoundary` component you can reset any query errors within the boundaries of the component.
- [ ] https://tanstack.com/query/v5/docs/react/reference/QueryErrorResetBoundary