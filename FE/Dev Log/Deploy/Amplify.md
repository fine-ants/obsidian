# Amplify 배포

## Table of Contents
- [[#Redirect and Rewrites 설정]]

## Redirect and Rewrites 설정


- Source Address
	```
	</^[^.]+$|\.(?!(css|gif|ico|jpg|js|png|txt|svg|woff|woff2|ttf|map|json|webp)$)([^.]+$)/>
	```
- Target Address
	- `/index.html`
- Type
	- `200 (Rewrite)`