📌 QueryDSL 주의사항
=

## 1.
N : 1 : N 연관관계에서  
1 쪽에서 2개 이상의 Collection 을 한꺼번에 fetchJoin 하려고 하는 경우 실패한다.  
왜냐하면 DB 관점에서 N * N 의 카타시안 곱 rows 가 생성되기 때문이다.

![image](https://user-images.githubusercontent.com/19279163/160080103-b852ad31-5bf2-4082-aa90-91a6b726a5cf.png)


위와 같은 에러가 뜨면서 실패하게 된다.

## 2.
1 : N 연관관계에서
1 쪽에서 N 쪽으로 Collection FetchJoin 을 하면서 (한 개의 Collection)  
동시에 pagination 까지 하려고 하면 Warning 이 발생한다. (하면 안된다.)

![image](https://user-images.githubusercontent.com/19279163/160081778-c8f3af4d-6539-49f3-804c-0e478eb2144b.png)

위와 같은 warning이 뜨는데, 일단 원하는 결과는 만들어주긴 한다.  
모든 결과를 다 불러와서 애플리케이션 단에서 pagination 을 하고 결과를 반환하기 때문이다.  
절대로 사용해서는 안되는 방법이다.
