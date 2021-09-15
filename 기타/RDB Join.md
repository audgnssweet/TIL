## RDB Join

    RDB 에서 join 또는 결합 구문은 한 데이터베이스 내의 여러 테이블의 레코드를 조합하여 하나의 열로 표현한 것이다.
    즉, 여러 개 테이블을 합쳐서 하나의 테이블이 되는 것을 뜻한다.

    RDB는 중복을 싫어하기 때문에 테이블 설계를 할 때 정규화를 하고, 데이터를 불러올 때는 잘 정규화된 테이블끼리
    관계를 이용하여 Join해서 가져온다.

    MySQL의 기본적인 Join template는 아래와 같다.
    SELECT ~~~ FROM TABLE_A (AS A) (INNER|LEFT|RIGHT) JOIN TABLE_B (AS B) ON A.FK = B.PK
    
    on 뒤에 조건을 명시(하냐마냐)하고, on 과 where의 동작 방식은 다르다. 
    a. inner join에서의 on과 where은 같은 결과를 만들어내고 (동작 방식은 다른데)
    b. outer join에서의 on과 where은 다른 결과를 만들어낸다. (동작 방식도 다르다)

    [중요] on 과 where 의 동작방식
    명확하게 기억하면 된다.
    on 은 join을 하냐 마냐
    where 은 필터링
    끝.

    on 절이 아예 없으면 카테시안곱을 만들어내기 때문에 주의해야한다. 일부러 카테시안곱을 만들어내는 것 (세타조인 - cross join)
    이외에는 반드시 on에 조건을 명시해야한다. (세타조인은 from절에서 해주면 되기에 on을 빼는 경우는 없다고 봐도 된다)
    
    연관관계 없는 join (FK가 아닌 다른조건으로 join)은 두가지 경우다.
    1. 카테시안 곱으로 모든결과를 만들어서 where로 필터링할 때 (from에 여러개 나열)
    2. 연관관계 없는 join이지만 outer join (left join) 을 활용해서 원하는 결과값을 얻어내고 싶을 때 (on절에서 FK 사용 안함)
    
    on 에 and로 여러개 조건을 명시할 수 있다.

    기본적으로 on의 조건은 PK와 FK 로 한다. (연관관계 없는 join - 카테시안곱에서 결과를 걸러내야하는 경우 제외)

### 예제

![image](https://user-images.githubusercontent.com/19279163/133459303-7c805673-764a-49c7-a147-8fc39c35ed65.png)

    테이블은 위와 같다.

![image](https://user-images.githubusercontent.com/19279163/133465836-03a55553-9f64-4eae-a5ca-a39bc5391506.png)

    inner join vs. left join (on m.team_id = t.team_id)
    inner join은 두 테이블을 비교하면서 team_id가 반드시 일치해야만 join을 실행하여 결과값으로 만든다.
    left join은 left table이 member이기 때문에 일단 member table을 다가져와서 결과로 만든다.
    그리고 on 조건에서 join을 하냐 마냐로 나뉘기 때문에 id가 일치하는 멤버 옆에 team을 join해서 붙여준다.
    null은 당연히 join이 안되는데, left join 특성상 이미 끌어왔기 때문에 join에 실패하면 null로 채운다.

    [정리]
    inner join 은 일단 왼쪽 테이블을 전부 가져오고, on 조건에 맞춰서 오른쪽 붙인 다음에 join에 실패한 row는
    결과값에서 제외시킨다.
    left join은 일단 왼쪽 테이블을 전부 가져오고, 거기에 on 조건에 맞춰서 오른쪽 테이블을 붙인 후,
    못붙이는 곳은 null로 만들어 반환한다.

![image](https://user-images.githubusercontent.com/19279163/133459491-33b17697-c776-49bd-9c1d-3eec70833d56.png)
![image](https://user-images.githubusercontent.com/19279163/133459510-8a53bbdd-9047-4cdf-bbac-86926b52e018.png)
![image](https://user-images.githubusercontent.com/19279163/133467246-7de83643-3e30-4f31-ac6a-2919c6ed808a.png)
![image](https://user-images.githubusercontent.com/19279163/133478883-0cd9d7f4-0271-44d6-bd88-b3da717949ff.png)

    on 뒤에 조건이 없으면 TABLE_A의 모든 Column과 TABLE_B의 모든 Column을 합쳐서
    총 A_ROW * B_ROW 의 개수가 생긴다. (위에서는 10개) = 카테시안 곱
    또한 조건이 없는 join은 inner와 outer join의 결과값이 같은 것을 확인할 수 있다.
    (마지막 두 개의 처럼 theta join을 구현할 수 있다. - cross join)

    그러므로 반드시 조건을 명시해야 한다.

![image](https://user-images.githubusercontent.com/19279163/133467897-885e803f-8f64-48f3-bd74-931a1c3379bf.png)

    위에서의 설명과 같이, inner join에서는 on과 where의 역할이 같음을 알 수 있다.
    (동작 방식은 다른데 결과가 같다는 것)

![image](https://user-images.githubusercontent.com/19279163/133468286-a9976ae9-2477-46d1-8696-176c4cf4cfa6.png)

    하지만 outer join에서는 명백히 다른 결과를 보여준다.

    마지막에 where은
    -> 일단 on 조건을 기준으로 join하기 때문에, PK, FK 기준으로 left join을 한 후 where로 필터링한다.

    on and 는
    -> join을 하냐 마냐의 조건이 (on 조건1 and 조건2) 일때 조건1, 조건2를 만족해야 join을 하고,
    left join이기 때문에 join에 실패한 row들은 null로 채우는 것.

    [TMI]
    일단 결론부터 말하면, outer join에서 PK와 FK 가 있다면 반드시 둘의 일치조건을 먼저 걸고, 그다음에 and로 조건을 추가하는 방식이
    어떤 의도이든 맞아들 확률이 높다. 아니면 inner join으로도 충분히 해결될 확률이 높다.

    on 에 FK 조건을 넣지 않는다면, 연관관계 없는 조인이다.

![image](https://user-images.githubusercontent.com/19279163/133451672-714a93a2-fc4c-4e58-8abc-67688db566b3.png)

    위 쿼리는 on절이 없으므로 카타시안 곱으로 테이블을 생성한 뒤에 조건에 맞춰 필터링한 것.
    연관관계가 없는 join이다.

![image](https://user-images.githubusercontent.com/19279163/133471878-50a97739-a2de-4942-849a-3a170cbce42a.png)

    동작 방식은 이렇다. 
    먼저 left join이므로 member의 모든 레코드를 전부 가져와서 왼쪽에 붙인다.
    이제 join를 하는데, 그 조건이 m.team_id=1 인 경우이다. (member, left table에 대한 조건)
    우리의 레코드에서 m.team_id=1 인 경우는 멤버 1,2 둘이 있다.
    그런데 이외 조건 (team, right table 에 대한 조건) 이 없으므로 무지성으로 m.team_id=1인 멤버 오른쪽에
    모든팀을 한번씩 다갖다가 join시켜버린다.
    그래서 멤버 1,2는 각각 팀 3번까지의 레코드, 3,4,5는 한개 (join자체를 하지 않으므로) 가 되는 것이다.

    마찬가지로 연관관계가 없는 join이다.

![image](https://user-images.githubusercontent.com/19279163/133451736-9a666fe7-d879-4c0f-9179-d1f4fe16817e.png)

    동작 방식은 이렇다.
    먼저 left join이므로 member의 모든 레코드를 전부 가져와서 왼쪽에 붙인다.
    이제 join을 하는데, 그 조건이 t.team_id=1인 경우다. (team, right table에 대한 조건)
    우리의 레코드에서 right table에 team_id=1인 경우는 team1밖에 없다.
    이를 왼쪽에 이미 붙여져있는 모든 member record (left join이므로) 에 붙인다.

    마찬가지로 연관관계가 없는 join이다.
