# **Long Polling**

Long Polling은 기존의 HTTP Polling 방식을 개선한 기법으로, 클라이언트가 서버에 요청을 보내고, 서버에 새로운 데이터가 준비되면 즉시 응답을 보내는 방식입니다.

클라이언트는 서버로부터 응답을 받는 즉시 다시 요청을 보내어 연결을 유지합니다.

### 장점

- 구현이 쉽다
- 거의 모든 브라우저에서 호환이 가능하다

### 단점

- 매번 새로 서버에 연결하는 것은 서버에 부담이 된다.
- 연결을 오랫동안 연결해 두어야 하기 때문에 동시에 많은 클라이언트를 처리할 경우 메모리와 스레드 사용에 부하를 줄 수 있다.
- 빈번한 `요청 → 응답` 사이클은 클라이언트의 많은 리소스를 소모하며 이는 모바일 기기에서 배터리 소모에 영향을 줄 수 있다.

### 선택을 고려할 상황

- 단순하고 낮은 횟수의 실시간 요청
- 오래된 브라우저를 지원해야 하는 경우

<hr>
# **Websocket**

WebSocket은 웹상에서 양방향 통신을 가능하게 하는 기술입니다.

HTTP와 달리, WebSocket은 연결을 맺은 후 지속적인 양방향 통신을 위한 전용 채널을 제공합니다.

### 장점

- 지연 시간을 최소화하며 실시간으로 데이터를 교환할 수 있다.
- 연결이 한 번만 맺어지고, 그 후로는 지속적인 데이터 교환을 위해 재연결할 필요가 없고 HTTP에 비해 오버헤드가 적어 효율적이다
- 클라이언트와 서버 양쪽에서 동시에 데이터를 보낼 수 있다.
- 클라이언트와 서버 모두 부담이 적다.

### 단점

- 모든 브라우저에서 지원하지 않는다.
- 종료된 연결을 자동 복구하지 않는다.
- websocket 전용 인프라를 서버에서 따로 구현해야 할 수 있다.

### 선택을 고려할 상황

- 프로젝트의 지원할 브라우저에 따라서 선택
- 클라이언트와 서버 간의 연결을 지속적으로 유지하기 때문에, 여러 연결을 효율적으로 관리하고 부하를 분산하기 위한 추가적인 서버 측 로직이 필요

<hr>
# **Websocket - stomp**

Websocket 은 기본적으로 텍스트와 바이너라 타입의 메시지만을 양방향으로 주고받을 수 있는 프로토콜 입니다.

그러나, 그 메시지를 어떤식으로 주고받을지는 사실 따로 정해진 것이 없습니다.

STOMP는 메시징 교환의 형식과 규칙을 정의하는 단순한 텍스트 기반 프로토콜입니다. STOMP를 사용하면 클라이언트와 서버 간에 메시지를 교환할 때 명확한 구조를 가지고 통신할 수 있습니다.

### 장점

- 클라이언트와 서버 간의 메시지 교환을 위한 형식이 정의되어 있다.
- 헤더와 바디를 가지며, 이를 통해 메시지에 대한 메타데이터를 명확하게 전달할 수 있다
- 여러 언어를 지원한다

### 단점

- WebSocket 메시지보다 크기가 크기 때문에, 통신에 더 많은 대역폭을 사용할 수 있으며, 소프트웨어 처리 오버헤드도 증가할 수 있다
- WebSocket에 비해 STOMP는 추가적인 복잡성을 도입된다.
    - 단순한 양방향 통신이 필요한 경우, STOMP의 이 기능은 과도할 수 있다.
- 메시지 브로커를 거치기 때문에 WebSocket에 비해 낮은 지연 시간을 제공할 수 없다.

### 선택을 고려할 상황

- 서로 다른 언어와 플랫폼을 사용하는 클라이언트 간의 통신이 필요한 경우
- 복잡한 메시징 패턴이나 트랜잭션, 메시지 선택, 메시지 순서, 메시지 확인 등과 같은 고급 기능을 필요로 한 경우

<hr>
# **Server-Sent Events(SSE)**

SSE는 서버가 클라이언트에 데이터를 사전에 푸시할 수 있도록 클라이언트와 서버 간의 장기 통신을 설정하는 방법입니다.

이는 단방향입니다. 즉, 클라이언트가 요청을 보내고 나면 동일한 연결을 통해 새 요청을 보내는 기능 없이 응답만 받을 수 있습니다.

### 장점

- HTTP를 기반으로 하기 때문에 간단하다
- 클라이언트가 연결이 끊어진 경우 자동으로 서버에 재연결을 시도한다.
- 연결을 한번만 한다.

### 단점

- 모든 브라우저에서 지원하지 않는다
- 동시 연결에 대한 브라우저와 서버의 제한이 있으며, 이는 동시에 많은 연결을 관리해야 하는 대규모 애플리케이션에는 문제가 될 수 있다.
- HTTP/2에서는 SSE의 성능 이점이 크게 줄어들 수 있으며, HTTP/2의 스트림을 활용한 방식을 고려할 수도 있다

### 선택을 고려할 상황

- 서버에서 클라이언트로의 단방향 통신
- 브라우저의 지원 여부
- 서버가 유저의 수를 감당 가능한지

<hr>
# FineAnts에 맞는 방법은?

- long polling
    - 주식 장이 열려 있다면 약 5초에 1번 서버로 부터 실시간으로 데이터를 받기 때문에 적절하지 못한 것 같다.
- Websocket
    - 가장 평범한 방법이라고 생각이 된다.
    - 과거에는 explorer에서 지원하지 않아 고려할 부분이 있었지만 이제는 아니다.
    - 모든 브라우저에서 지원하기 때문에 정말 폭 넓은 브라우저를 지원할 계획이라면 채택할 이유로 충분해 보인다.
- Websocket - stomp
    - 프론트엔드 기준으로는 stomp의 필요 가치를 못 느끼겠다.
        - 복잡한 메시징 패턴을 사용하지 않는다.
        - 언어가 다른 클라이언트 끼리의 통신을 하지 않는다
    - 백엔드의 입장은 잘 모르겠다..
- Server-Sent Events
    - 우선 모든 브라우저를 지원하지 않는다면 가장 적절해 보인다.
        - Deno 브라우저를 지원하지 않는다.
        - [https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events)
    - 실시간으로 서버와의 통신이 아닌 단방향으로 서버에게 받기만 하기 때문에 적절하다 생각이 든다.

# 참고

[https://www.karanpratapsingh.com/courses/system-design/long-polling-websockets-server-sent-events](https://www.karanpratapsingh.com/courses/system-design/long-polling-websockets-server-sent-events)
[](https://velog.io/@msung99/%EC%9B%B9%EC%86%8C%EC%BC%93%EA%B3%BC-STOMP%EB%A5%BC-%ED%86%B5%ED%95%9C-%EC%8B%A4%EC%8B%9C%EA%B0%84-%ED%86%B5%EC%8B%A0-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0)[https://velog.io/@msung99/웹소켓과-STOMP를-통한-실시간-통신-이해하기](https://velog.io/@msung99/%EC%9B%B9%EC%86%8C%EC%BC%93%EA%B3%BC-STOMP%EB%A5%BC-%ED%86%B5%ED%95%9C-%EC%8B%A4%EC%8B%9C%EA%B0%84-%ED%86%B5%EC%8B%A0-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0)

[https://stackoverflow.com/questions/40988030/what-is-the-difference-between-websocket-and-stomp-protocols](https://stackoverflow.com/questions/40988030/what-is-the-difference-between-websocket-and-stomp-protocols)