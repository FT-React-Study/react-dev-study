# Suspense
먼저 글 두 개를 소개하겠습니다.
첫 번째 글은 제가 처음 Suspense를 사용했을 때 많이 참고한 카카오페이의 블로그이고, 두 번째 글은 제가 Next.js 환경에서 Suspense를 사용하면서 겪었던 어려움, 삽질... 그리고 Suspense와 Next.js를 조금씩 뜯어본 내용을 담았습니다.
- [무조건 스켈레톤 화면을 보여주는 게 사용자 경험에 도움이 될까요?](https://tech.kakaopay.com/post/skeleton-ui-idea/)
- [SSR 환경에서 Suspense 도입 시 발생할 수 있는 문제와 해결 방법](https://velog.io/@chchaeun/Skeleton-UI%EB%A1%9C-%EC%82%AC%EC%9A%A9%EC%9E%90-%EA%B2%BD%ED%97%98-%EA%B0%9C%EC%84%A0%ED%95%98%EA%B8%B0-with-Suspense)

이번에 Suspense를 다시 공부하고 토스 과제 전형도 진행하면서 토스의 @suspensive/react-query에 대해서 공부하게 됐는데 제가 위 블로그를 작성하면서 고민했던 지점을 해결해주어서 흥미로워서 공유해봅니다.

### Suspense / SuspenseQuery

- Suspense: 자식 컴포넌트에서 발생하는 비동기 흐름을 감지하여 Promise의 상태에 따라 fallback UI 혹은 children을 렌더링합니다.
- 우리는 UI와 데이터를 분리하기 위해 컴포넌트로 데이터를 보내주는 구조를 사용합니다.
    
    ```tsx
    // useContractQuery.ts
    import { useQuery } from "@tanstack/react-query";
    import { getContractsAPI } from "remotes";
    
    export const useContractsQuery = () => {
      return useQuery({ queryKey: ["contracts"], queryFn: getContractsAPI });
    };
    
    // index.tsx
    export default function IndexPage() {
    	const { isLoading, data: contracts } = useContractsQuery();
    	
    	return (
    		<>{isLoading ? 
    				<Fallback/>
    			:
    				<ContractList
    					contracts={contracts}
    				/>
    		}<>
    	);
    }
    ```
    
- 여기서 Suspense를 적용하려면 데이터를 페칭해주는 컴포넌트가 필요합니다. 데이터 패칭을 하는 컴포넌트가 Suspense의 자식이어야 되기 때문입니다.
    
    ```tsx
    // index.tsx
    export default function IndexPage(){
    	return (
    		<Suspense fallback={<Fallback/>}>
    			<ContractListFetcher/>
    		</Suspense>
    	);
    }
    
    // ContractListFetcher.tsx
    export default function ContractListFetcher(){
    	const { data: contracts } = useContractsQuery();
    	
    	return (
    		<ContractList
    			contracts={contracts}
    		/>
    	);
    }
    ```
    
- `SuspenseQuery` 를 사용하면 패칭만을 위한 컴포넌트를 제거할 수 있고, queryOptions를 사용하여 useQuery hook을 작성하지 않아도 됩니다.
    - 반복되는 보일러 플레이트 코드가 크게 줄어드는 이점이 있습니다.
    - https://velog.io/@chchaeun/Skeleton-UI로-사용자-경험-개선하기-with-Suspense#난관-2-기존-컴포넌트의-재사용-구조-사용-불가
        - 위와 같은 코드를 작성하면서도 좋은 구조가 아닌 것 같아서 신경 쓰였는데(데이터와 컴포넌트의 과도한 결합, 하나의 query hook의 역할 과중), 같은 고민을 하고 해결책을 도출한 사람들이 있다는 게 존경할만 한 지점이라고 생각했습니다...
    
    ```tsx
    // queries.ts
    
    export const contractsQueryOptions = () =>
      queryOptions({
        queryKey: ["contracts"],
        queryFn: getContractsAPI,
    });
    
    // index.tsx
    
    export default function IndexPage(){
    	return (
        <Suspense>
          <SuspenseQuery {...contractsQueryOptions()}>
            {({ data: contracts }) => (
              <ContractList
                contracts={contracts}
              />
            )}
          </SuspenseQuery>
        </Suspense>
    	)
    }
    ```
- 현석님과의 대화(Fetcher를 분리해야 하는 이유에 대해 설명이 부족한 것 같아 추가했습니다.)
    - 현석 님: tanstack query 사용 시 suspense 옵션을 true로 주는걸로 List/Fetcher 분리 안해도 되지 않나요? useQuery 캐싱을 사용하려면 어차피 클라이언트 사이드여야 할테니 server/client랑은 상관없을거고요
    - 저:
    데이터 페칭을 따로 분리할 필요는 없는데 저는 데이터랑 ui를 렌더링하는 부분이랑 분리해야 책임이 나눠져서 좀 더 좋은 구조가 된다고 생각했어요! 
        예를 들어서 
        ```
        <input name="title"/>
        <input name="content"/>
        ```
        라는 컴포넌트가 있는데, (서버) 데이터가 있는 경우 value를 채워주고 싶다면 해당 컴포넌트에서 페칭하는 것보다
        ```
        const {error, data} = use___Query();
        
        <input name="title" value={error.status === 404 ? "": data.title}/>
        <input name="content" value={error.status === 404 ? "": data.content}/>
        ```
        
        부모에서 내려주는 게 좀 더 처리하기 좋다고 생각했거든요. (ErrorBoundary를 사용하면 또 error가 전역에서 캐치가 돼버리니까 따로 핸들링해줘야되고)
        ```
        const Component = ({data}) => {
          <input name="title" value={data.title || ""}/>
          <input name="content" value={data.content || ""}/>
        }
        ```
        
        그래서 항상 렌더링하는 부분에서 데이터를 페칭하지 말고 위에서 받아오는 형식으로 작성했는데 그러다보니까 원래 page에서 query를 불러왔다면, 이제는 suspense를 page에 작성해줄 거기 때문에 한단계 depth를 추가할 수밖에 없게 돼버린...
        
        [https://suspensive.org/ko/docs/react-query/SuspenseQuery](https://suspensive.org/ko/docs/react-query/SuspenseQuery)
        여기에도 비슷하게 설명이 돼있더라구요! 
    - 현석 님: 아 서스펜스를 위해 그렇게 해야한다~ 정도로 적어주신것 같았어서요 저도 페칭/로직파트와 View를 분리하는 VAC 패턴에 대해 알고는 이 패턴이 좋은 방식 같다고 생각하고 있습니다. 이전의 Container 패턴과 유사한데 해당 패턴이 테스트 관점, 역할 분리 관점에서 좋아보이는것 같습니다.


- Suspense로 여러 개의 쿼리를 함께 처리하고 싶으면 SuspenseQueries를 사용하면 됩니다.
    - https://suspensive.org/ko/docs/react-query/SuspenseQueries
 
## React Query - placeholderData
어떤 리스트가 있고, 필터를 통해 해당하는 데이터만 서버에 요청해서 가져오는 코드가 있다고 가정합시다. 초기에 Suspense를 적용하면 리스트가 불러와질 때까지 로딩 인디케이터가 뜹니다. 하지만 그 다음 필터를 변경했을 때 또 로딩 인디케이터가 뜨게 되면 사용자는 리스트 -> 인디케이터 -> 새 리스트라는 화면의 변화를 겪어야되고 렌더링 경험이 악화될 수 있습니다.
이러한 불편을 해결하기 위해서 useDeferredValue라는 React에서 제공하는 훅을 사용할 수 있는데요. React Query를 통해 손쉽게 이것을 구현할 수 있습니다.
queryOptions의 placeholderData 옵션은 데이터가 없을 때 해당 데이터로 대신해주는 placeholder 역할을 하는 data인데요, 여기에 keepPreviousData라는 값을 넣어줄 수 있습니다. 다음과 같이요.
```
import { keepPreviousData } from '@tanstack/react-query'

useQuery({
  queryKey: ["post", id],
  queryFn: () => fetchPost(id),
  placeholderData: keepPreviousData
})
```

그러면 새로운 데이터를 패칭하는 동안 이전에 쿼리에서 가지고 있던 데이터를 유지하고 있어 리스트 -> 새 리스트로 매끄러운 화면 전환이 가능합니다.
이것도 최근에 새로 알게 된 기능이라 재미있어서 추가로 넣어봤어요.
