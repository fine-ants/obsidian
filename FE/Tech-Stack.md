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
```
type Props = {}

export default function App({}: Props) {}
```
- Styled Components
```
export default function App({}: Props) {}

const StyledApp = styled.div``;
```

```
${({ theme: { opacity } }) => opacity.hover};
```
- Function Naming
```
ON TARGET EVENT

Ex: onDropdownClick
```
- Function Declarations
	- Component는 `function`
	- 내부 함수들은 const
```
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
