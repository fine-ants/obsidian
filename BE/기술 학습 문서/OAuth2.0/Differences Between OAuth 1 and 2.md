OAuth2.0은 OAuth1.0을 처음부터 완전히 다시 작성한 것으로, 전체 목표와 일반 사용자 경험만을 공유합니다. OAuth2.0은 OAuth 1.0 또는 1.1과 역호환(backwards compatible)되지 않으므로 완전히 새로운 프로토콜로 간주해야 합니다.

OAuth 1.0은 주로 Flickr의 authorization API와 구글의 AuthSub의 두가지 독점 프로토콜을 기반으로 하고 있습니다. OAuth 1.0이 되었던 작업은 당시 실제 구현 경험을 바탕으로 한 최고의 솔루션이었습니다. 몇년동안 많은 회사들이 OAuth 1.0 API를 구현하고 많은 개발자들이 API를 사용하기 위해서 코드를 구현하면서 커뮤니티는 OAuth 1.0 프로토콜이 사람들에게  구현하는데 어렵다는 것을 알게 되었습니다. 일부 특정 지역에서는 API의 기능들을 제한하거나 구현하기가 어려워 개선이 필요하였습니다. 

OAuth 2.0은Yahoo!, Facebook, Salesforce, Microsoft, Twitter, Deutsche Telekom, Intuit, Mozilla 그리고 Google을 포함한 광범위한 기업과 개인 간의 수년간의 논의한 것을 나타냅니다.

이 장에서는 OAuth 1.0과 OAuth 2.0 사이에 주요 차이점과 배경 동기에 대해서 다룹니다.만약 여러분들이 OAuth 1.0에 익숙하다면, OAuth 2.0에서의 주요 변경점들을 빠르게 이해하는 좋은 포인트일 것입니다.

### Terminology and Roles
OAuth 2.0이 4가지 역할들을 정의할 때(client, authorization server, resource server, resource owner), OAuth 1.0은 이러한 역할들에 대해서 다른 용어들을 사용합니다. OAuth 2.0의 "client"는 "consumer"로 알려져 있고, "resource owner"는 간단하게 "user", "resource server"는 "service provider"로 알려져 있습니다. OAuth 1.0은 또한 resource server와 authorization server의 역할을 명확하게 구분짓지 않습니다.

"two-legged"과 "three-legged" 용어는 Client Credentials Grant Type과 Authorization Code Grant Type과 같은 Grant Type이라는 용어로 대체되었습니다.

### Summary
OAuth 2.0은 OAuth 1.0과는 호환되지 않는 완전히 다른 프로토콜입니다. OAuth 1.0 프로토콜은 개발자들이 구현하는데 어렵고 일부 특정 지역에서는 API 기능을 제한하거나 구현하기가 어려워 개선이 필요하였습니다. OAuth 1.0의 consumer는 OAuth 2.0의 client, user는 "resource owner"로, "service provider"는 "resource server"로 역할이 알려져 있습니다. OAuth 1.0은 resource server와 authorization server의 역할을 명확하게 구분짓지 않지만 OAuth 2.0은 명확하게 구분짓습니다. 마지막으로 
