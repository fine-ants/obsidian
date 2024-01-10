# TanStack Query and ErrorBoundary

## ErrorBoundary
- React는 `componentDidCatch(error, info)` API를 통해 render 과정에서 일어나는 에러를 잡을 수 있는데, 이는 현재 class component에만 가능하다.
- ErrorBoundary class component을 만들거나 `react-error-boundary`와 같은 library를 사용하면 된다.
### Reference
- https://react.dev/reference/react/Component#componentdidcatch
- https://react.dev/reference/react/Component#componentdidcatch-caveats
- https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary

## ErrorBoundary with TanStack Query
- 보통의 경우 아래 코드처럼 구조를 ErrorBoundary와 Suspense를 사용하게 된다.
```tsx
<ErrorBoundary FallbackComponent={ErrorFallback}>
	<Suspense fallback={SuspenseFallback}>
		<MyComponent />
	</Suspense>
</ErrorBoundary>
```
- TanStack Query에서 제공하는 `useSuspenseQuery`를 사용하여 React의 `Suspense`와 함께 TanStack Query를 사용할 수 있게 된다.
- 이 `useSuspenseQuery`는 React의 `Suspense`을 지원하기 위해 내부 cache을 사용하고 이 cache는 `gcTime` 설정과 별개로 운영인된다. 그러므로 `gcTime`을 `0`으로 설정해도 error 응답 마저 필연적으로 cache가 된다.
- 그렇다면, error 발생 후 재시도를 가능하게 하려면 ErrorFallback component가 render되면서 해당 query에 재시도 의도를 알려줘야 한다.
- TanStack Query는 `QueryErrorResetBoundary` component을 통해 해당 boundary 내에 있는 component에서 query의 error를 reset할 수 있도록 한다.
```tsx
<QueryErrorResetBoundary>
	{({ reset }) => (
		<ErrorBoundary onReset={reset} FallbackComponent={ErrorFallback}>
			<Suspense fallback={SuspenseFallback}>
				<MyComponent />
			</Suspense>
		</ErrorBoundary>
	)}
</QueryErrorResetBoundary>
```

### Reference
- https://tanstack.com/query/v5/docs/react/reference/QueryErrorResetBoundary
