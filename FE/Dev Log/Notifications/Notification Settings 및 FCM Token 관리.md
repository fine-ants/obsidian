# Notification Settings 및 FCM Token 관리

## 상황
- 사용자가 `NotificationSettingsDialog`에서 `browserNotify`, `maxLossNotify`, `targetGainNotify`, `maxLossNotify`, `targetPriceNotify`을 토글을 조작할 수 있다.
- 조작 후, 저장 버튼을 누르게 되면, 1) 해당 상태 값을 서버에 반영하는 요청을 보내고, 2) 해당 상태 값들에 따라 FCM token을 등록/해제하는 요청을 보내야한다.
```ts
const onSubmit = async () => {
	if (isDisabledButton) return;

	const body = {
		browserNotify: newBrowserNotify,
		maxLossNotify: newMaxLossNotify,
		targetGainNotify: newTargetGainNotify,
		targetPriceNotify: newTargetPriceNotify,
		fcmTokenId,
	};
	
	try {
		// 1) 토글 상태값 반영
		await mutateAsyncNotificationSettings(body);

		// 2) 토글 상태값에 따라 FCM Token 등록/해제
		if (newBrowserNotify && !fcmTokenId) {
				const newFCMTokenId = await onActivateNotification();
			if (newFCMTokenId) {
				onSubscribePushNotification(newFCMTokenId); // user context update
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
				toast.success("알림 설정을 해제했습니다");
			} catch (error) {
				toast.error("알림 설정을 해제하는데 문제가 발생했습니다");
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
	- Ex: 