
<hr>

## 문제

종목에 대한 검색요청 시 매 input에 새로운 value가 들어올때마다 사용되지않는 이전 값들이 inactive 되어있는 상태의 쿼리로 캐싱되어있으므로 사용자가 많거나 검색어가 빠르게 변할 때, 혹은 그 둘 다일 경우 메모리 누수가 생길 확률이 높아진다. ex) 현재 input창에 '삼성전자'가 입력되어있다면 입력되기까지에 값들인 'ㅅ', '사', '삼', '삼ㅅ' 분리된 검색어들이 inactive된 상태로 캐시되어있다. 

**그렇다면 TanStack Qeury는 inactive된 value들을 알아서 처리해주는가?**

그렇다.TanStack Query는 사용되지 않는 쿼리 데이터를 내부적으로 추적한다. inactive된 쿼리는 일정 시간이 지난 후 자동으로 가비지 컬렉션될 수 있도록 `gcTime`(이전 버전에서는 cacheTime)이라는 설정을 제공한다. `gcTime`은 쿼리가 비활성 상태로 남아있는 시간을 정의하며, 이 시간이 지나면 쿼리 결과가 메모리에서 제거된다. 기본값은 5분이다. 하지만 사용자가 연속해서 많은 수의 쿼리를 생성하는 경우, 가비지 컬렉션이 발생하기 전에 많은 양의 메모리를 사용할 수 있으며, 이는 성능 저하나 메모리 누수로 이어질 수 있다. 이를 방지하기 위한 방법은 무엇일까?


<hr>

## 해결방안 


### 1. 디바운스 적용

##### 장점

디바운스 기법은 사용자가 입력을 마친 후 지정된 시간이 지나야만 완성된 검색어를 통해 쿼리 요청을 보내게 함으로써 inactive 상태로 캐시될 쿼리의 수를 현저히 줄일 수 있다. 이는 네트워크 부하를 경감시키고, 서버의 과부하를 방지하며, 메모리 사용을 최적화하는데 기여한다.

- 불필요한 API 요청 감소
- 서버 부하 경감
- 메모리 사용량을 최적화
##### 단점

사용자 경험(UX) 측면에서 디바운스는 상호작용의 즉시성을 저해할 수 있다. 대다수 사용자는 입력과 동시에 결과를 기대하며, 특히 자음 하나하나에 반응하여 즉각적인 피드백을 제공하는 검색 인터페이스에 익숙한 경우, 디바운스로 인한 지연은 불편함으로 느껴질 수 있다. 이러한 문제를 완화하기 위해 사용자에게 현재 검색이 진행 중임을 알리는 시각적 신호(예: 로딩 인디케이터)를 제공하는 것이 중요하다.

- 즉시 반응하지 않는 UX
- 사용자가 입력 반응 지연을 불편하게 느낄 수 있음
##### 보완점

디바운스 기능을 적용할 때는 사용자에게 입력이 처리되고 있음을 알리는 시각적인 피드백을 제공하여 UX를 개선할 수 있다. 예를 들어, 입력창 아래에 '검색 중...'과 같은 메시지나 로딩 스피너를 표시하여 사용자가 시스템이 반응하지 않는 것이 아니라 검색 결과를 가져오고 있음을 인지할 수 있도록 해야 한다. 또한 디바운스의 지연 시간을 적절히 조정하여 사용자가 불필요한 대기 시간 없이 최대한 빠른 피드백을 받을 수 있도록 하는 것이 중요하다.


### 2. staleTime과 gcTime의 설정을 통한 캐시 관리 전략

**`staleTime`과 `gcTime` 설정 이전에 `TanStack Query`는 실제로 `inactive` 되어있던 쿼리의 데이터를 `fetch`해오는가? 새로운 검색어를 입력하고 쿼리 요청을 보낼 때 이전의 입력되어있던 검색어들의 쿼리 키들은 `inactive`한 상태로 캐싱된다. `gcTime`이 끝나기전에 `inactive`한 상태로 캐싱되어있는 쿼리 키들의 검색어를 입력했을 때 해당 쿼리 키의 값을 `active`한 상태로 재활용하는가?

1. 위와 같은 고민이 생겼던 이유

`staleTime`을 길게 설정하면, 그 시간 동안 데이터는 `fresh` 상태로 유지된다. 이 상태는 최근에 데이터를 가져왔고, 아직 변할 가능성이 없다는 것을 의미한다. 데이터가 `fresh` 상태일 때는 동일한 쿼리키로 요청을 해도 새로운 네트워크 요청을 하지 않고 캐시된 데이터를 그대로 쓴다.
`fresh`라는 건 새로 데이터를 요청했다는 걸 직접적으로 나타내는 게 아니라, `staleTime` 내에 성공적으로 데이터를 받아온 상태를 말한다. 그래서 `staleTime` 설정은 데이터가 얼마나 자주 바뀔 수 있는지 추정해서 조정해야 한다.
이 시간 설정은 데이터가 `stale`로 바뀌기 전까지 얼마나 지속될지 정해주고, 그 시간 동안은 데이터가 바뀌지 않았다고 보고 추가적인 네트워크 요청 없이 바로 데이터를 쓸 수 있다. 데이터가 `stale` 상태가 되면, TanStack Query는 캐시된 데이터를 먼저 보여주면서도 배경에서 데이터를 새로 고치기 위해 네트워크 요청을 시작한다.
`inactive`한 쿼리키의 데이터를 다시 요청할 때 `staleTime`이 아직 안 지났으면 쿼리는 여전히 `fresh` 상태로 간주되어 새로운 네트워크 요청 없이 캐시된 데이터를 재사용한다. `staleTime` 설정할 때 애플리케이션에서 데이터 변동성을 잘 판단해서 설정한다.

2. 고민을 해결한 검증

`staleTime`을 5초로 설정하고, '생활'을 입력하는 과정에서 생성되는 각각의 쿼리는 최초 응답 이후 5초 동안 `fresh` 상태로 유지된다. 만약 '생활'의 마지막 '활'을 입력한 후 5초가 지난 시점에서 새로운 데이터를 받기위해 쿼리 요청하지 않는다면, 해당 쿼리의 데이터는 `stale` 상태가 되고 자동으로 `fresh`한 상태로 업데이트하기 위해 새로운 네트워크 요청을 보낸다.
이후 동일한 '생활'을 재입력할 때, 이전 쿼리들이 아직 `gcTime` (가비지 컬렉션 타임) 기간 내에 있다면, TanStack Query 라이브러리는 이전에 캐싱된 데이터를 재활용하고, 해당 쿼리들은 다시 `active` 상태가 된다. 만약 쿼리 데이터가 `stale` 상태인 경우, TanStack Query는 백그라운드에서 새로운 데이터를 가져오기 위한 네트워크 요청을 자동으로 시작할 수 있다.  (`stale`한 상태이므로 다시 `fresh` 상태로 변경시키기 위해 ) `staleTime`이 지나지 않았다면 자동으로 `fresh`한 상태가 된다.

**`fresh` 상태는 `active` 상태를 포함하고있는 것.**

이러한 행위는 애플리케이션의 데이터 신선도 요구사항과 사용자 상호작용에 따라 최적화되며, `staleTime`과 `gcTime` 설정은 사용자가 같은 데이터를 다시 요청할 때 네트워크 요청 없이 캐시된 데이터를 즉시 사용할 것인지, 아니면 새로운 데이터를 가져올 것인지를 결정하는 중요한 요소가 된다. 

<hr>
- A second instance of `useQuery({ queryKey: ['todos'], queryFn: fetchTodos })` mounts elsewhere.
    - Since the cache already has data for the `['todos']` key from the first query, that data is immediately returned from the cache.
    - The new instance triggers a new network request using its query function.
        - Note that regardless of whether both `fetchTodos` query functions are identical or not, both queries' [`status`](https://tanstack.com/query/latest/docs/react/reference/useQuery) are updated (including `isFetching`, `isPending`, and other related values) because they have the same query key.
    - When the request completes successfully, the cache's data under the `['todos']` key is updated with the new data, and both instances are updated with the new data.
<hr>

출처: https://tanstack.com/query/v4/docs/react/guides/caching#basic-example

**TanStack Query가 캐싱된 데이터를 재활용한다는 것을 확인했고, staleTime과 gcTime은 어떤 기준으로 짜야하는가?

- `staleTime`
	- 데이터가 얼마나 자주 변경될 것으로 예상되는지를 기준으로 삼아야한다.
	- 변동성이 높을수록 짧게 설정해서 최신화해주고, 낮을수록 길게 설정하여 불필요한 네트워크 요청을 줄여야한다.
	- 
