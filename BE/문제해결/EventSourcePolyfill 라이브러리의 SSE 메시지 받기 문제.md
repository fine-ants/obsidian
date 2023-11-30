## 상황
클라이언트에서 EventSourcePolyfill 라이브러리를 사용하여 SSE 응답 메시지를 받을때 메시지를 받을 수 없습니다.

```
const accessToken = localStorage.getItem("accessToken");

const eventSource = new EventSourcePolyfill(

`${BASE_API_URL}/api/portfolio/${id}/holdings`,

{

headers: {

Authorization: `Bearer ${accessToken}`,

},

}

);

  

eventSource.onmessage = async (e) => {

const res = await e.data;

// eslint-disable-next-line no-console

console.log(res);

};
```

## 원인
- 스프링 서버로부터 메시지 이름이 "sse event - myPortfolioStocks"라는 이름으로 응답하고 있는데 현재 코드상 메시지 이름이 "message"인 경우에만 받을 수 있습니다.

## 해결방법
다른 이름의 메시지로도 받을 수 있도록 방법이 필요합니다.
