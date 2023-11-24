
# react-testing-library ft. Jest

## Table of Contents
- [[#React Component doesn't forward its props to the Underlying HTML Element that is Rendered]]

## React Component doesn't forward its props to the Underlying HTML Element that is Rendered

### Example - ordinary html element
- This works fine.
```tsx
it("blah", () => {
    const { getByTestId } = render(
      <div data-testid="chico">I'm a div</div>
    );
	console.log(screen.debug(undefined, Infinity));

	expect(getByTestId("chico")).toBeDefined();
});
```

```
console.log
	<body>
		<div>
			<div data-testid="chico">I'm a div</div>
		</div>
	</body>
```

### Example - React Component
- `data-testid` attribute doesn't show when rendered.
```tsx
it("blah", () => {
    const { getByTestId } = render(
      <MyComponent data-testid="chico">I'm a div</MyComponent>
    );
	console.log(screen.debug(undefined, Infinity));

	expect(getByTestId("chico")).toBeDefined();
});
```

```
console.log
	<body>
		<div>
			<div>I'm a div</div>
		</div>
	</body>
```
