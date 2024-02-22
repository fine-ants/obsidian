# State Management

## User State
### 현재 User Context
```tsx
export const UserContext = createContext<{
	user: User | null;
	fcmTokenId: number | null;
	onSignIn: (signInData: SignInData) => void;
	onSignOut: () => void;
	onGetUser: (user: User) => void;
	onEditProfileDetails: (user: User) => void;
	onSubscribePushNotification: (fcmTokenId: number) => void;
	onUnsubscribePushNotification: () => void;
}>(
  // ...
)
```
#### 문제


