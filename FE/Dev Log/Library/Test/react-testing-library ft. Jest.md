
# 

## Table of Contents


## React Component doesn't forward its props to the Underlying HTML Element that is Rendered
- Ex: `data-testid` attribute doesn't show when rendered.


### Example - ordinary html element
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