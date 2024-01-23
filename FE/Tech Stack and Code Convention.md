# Tech Stack and Code Convention

## FE Tech Stack
- Vite
- React, React Router
- React Query
- TypeScript
- MUI, Styled Components
- Axios
- ESLint, Prettier
- Jest, testing-library/react
- MSW
- AWS Amplify

## Code Convention
- Props
```tsx
type Props = {}

export default function App({}: Props) {}
```
- Styled Components
```tsx
export default function App({}: Props) {}

const StyledApp = styled.div``;
```

```tsx
// Transient Props
${({ $isActive }) => $isActive ? "blue" : "black" };

// Design System
${designSystem.color.primary.blue500};
```
- Function Naming
```ts
// ON TARGET EVENT

const onDropdownClick = () => {}
```
- Function Declarations
	- Component은 `function`
	- 내부 함수들은 const
```tsx
export default function App({}: Props) {
	const hamsu = () => {};
	
	return ();
}
```
- Props
	- 1개일 때
		- Ex: `onDropdownClick={onDropdownClick}`
	- 2개 이상일 때
		- Ex: `{...{onDropdownClick, dropdownList}}`
