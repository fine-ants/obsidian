# Notification Settings 및 FCM Token 관리

## 상황
- 사용자가 `NotificationSettingsDialog`에서 `browserNotify`, `maxLossNotify`, `targetGainNotify`, `maxLossNotify`, `targetPriceNotify`을 토글을 조작할 수 있다.
- 조작 후, 저장 버튼을 누르게 되면, 1) 해당 상태 값을 서버에 반영하는 요청을 보내고, 2) 해당 상태 값들에 따라 FCM token을 등록/해제하는 요청을 보내야한다.
```ts
const onSubmit = async () => {
	if (isDisabledButton) return;

	const newSettingsBody = {
		browserNotify: newBrowserNotify,
		maxLossNotify: newMaxLossNotify,
		targetGainNotify: newTargetGainNotify,
		targetPriceNotify: newTargetPriceNotify,
		fcmTokenId,
	};
	
	try {
		// 1) 토글 상태값 반영
		await mutateAsyncNotificationSettings(newSettingsBody);

		// 2) 토글 상태값에 따라 FCM Token 등록/해제
		if (newBrowserNotify) {
			try {
				// FCM에 subscribe하고 server로 token 등록 요청
				const newFCMTokenId = await onActivateNotification();
				if (newFCMTokenId) {
					onSubscribePushNotification(newFCMTokenId); // user context update
				}
			} catch (error) {
				toast.error("FCM 토큰을 등록하는데 문제가 발생했습니다");
			}
		} else if (
			!newBrowserNotify &&
			!newMaxLossNotify &&
			!newTargetGainNotify &&
			!newTargetPriceNotify &&
			fcmTokenId
		) {
			try {
				// FCM에서 unsubscribe하고 server에 token 삭제 요청
				await onDeactivateAllNotifications(fcmTokenId);
				onUnsubscribePushNotification(); // user context update
			} catch (error) {
				toast.error("FCM 토큰을 해제하는데 문제가 발생했습니다");
			}
		}
		
		onClose();
	} catch (error) {
		toast.error("알림 설정을 변경하는데 문제가 발생했습니다");
	}
};
```

## 문제
- 만약, `mutateAsyncNotificationSettings`가 실패하면 FCM Token 로직이 실행이 안되기 때문에 이 부분은 괜찮다.
- 만약, `mutateAsyncNotificationSettings`가 성공했는데 `onActivateNotification` 또는 `onDeactivateAllNotifications`가 실패하면 토글 상태값과 FCM Token간의 차이가 생길 수 있다.
	- Ex: 토글을 다 `false`로 하고 서버에 변경을 성공적으로 반영했는데 FCM에서 token unsubscribe을 실패하면, 사용자는 알림 기능을 비활성화해놨지만 FCM token은 남아있게 된다. 서버에서 알림을 보내기 전에 토글 상태값을 확인하겠지만, 불필요하게 FCM 채널이 2달 동안 유효하게 남게된다.

## 해결 방법
### 1) Rollback
- `onDeactivateAllNotifications`가 실패했을 때, 사용자가 FCM 제거 과정을 다시 밟을 수 있도록 `browserNotify`를 `true`로 되돌리기.
```ts
try {
	await onDeactivateAllNotifications(fcmTokenId);
	onUnsubscribePushNotification();
} catch (error) {
	// Rollback
	await mutateAsyncNotificationSettings({
		...newSettingsBody,
		browserNotify: true, // `browserNotify`를 `true`로 되돌리기
	});
	toast.error("FCM 토큰을 해제하는데 문제가 발생했습니다");
}
```
#### 고민
- Rollback마저 실패한다면?
### 2) Retry


