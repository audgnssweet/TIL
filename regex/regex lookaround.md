<h2> REGEX lookaround </h2>

    lookaround 는 regex의 고급개념이다
    단 후방탐색은 지원하지 않는 언어가 있으므로, 전방 탐색만 다루겠다.

    positive는 일치, negative는 비일치이다

    전방 탐색
    (?=) 일치
    (?!) 비일치

    전방탐색이란 탐색 방향이 -> 인 것을 의미한다.
    원래 문자열을 탐색할 때의 방향과 같다.

---

    1. 긍정 전방탐색 (?=)

    (?=REGEX_1)REGEX_2

    a. 전방탐색이므로 처음부터 시작해서 -> 방향으로 REGEX_1 에 매칭되는 문자열이 나올 때까지 찾는다.
    b. 매칭되면, 매칭된 문자열을 마치 매칭되지 않은 것처럼 그곳부터 시작하여 REGEX_2 를 매칭시킨다.
    -> 말이 어려운데 이건 예시를 보면 된다.

    REGEX_1(?=REGEX_2)
    a. 전방탐색이므로 처음부터 시작해서 -> 방향으로 REGEX_2 에 매칭되는 문자열이 나올 때까지 찾는다.
    b. 매칭되면 REGEX_2 에 매칭된 문자열을 기준으로 그 앞에서만 REGEX_1 을 매칭시킨다.
    -> 마찬가지로 예제를 보면 된다.

    [예제]
    예제 문자열 https:www.naver.com

    a. REGEX_1(?=REGEX_2) 인 경우 - ".*(?=:)"
    매칭결과 https

    - REGEX_2 가 : 에서 매칭되었으므로, 그 기준으로 그 앞에서만 REGEX_1 인 .* 를 매칭시킨다.
    그러므로 https 만 매칭된 것이다.

    그래서 위와같은 형태의 lookaround는 원하는 패턴에서 특정 패턴만을 제외한 결과값을 받고싶을 때 사용한다.
    -> 마치 위에서처럼 프로토콜만 받고 싶을 때 : 까지 매칭시킨 후 이를 제외하고 뽑아낼 때

    b. (?=REGEX_1)REGEX_2 인 경우 - "(?=:).*"
    매칭 결과 :www.naver.com

    - REGEX_1이 : 에서 매칭되었으므로, 위에서의 설명대로 그 기준 그녀석부터 REGEX_2와 매칭시키기 때문이다.

    [추가]
    (?=REGEX_1)(?=REGEX_2)REGEX_3
    처럼 여러개의 조건을 선행시키는 것도 가능하다.

    2. 부정 전방탐색 (?!)

    (?!REGEX_1)REGEX_2
    이것도 위와 마찬가지인데, 부정조건인게 다른 점이다.

    마찬가지로 여러개 조건을 주는 것이 가능하다.

---

## 응용

    이를 응용하면, 문자열 중간에 특정 패턴이 들어가있는지를 확인할 수 있다.

    예를들어 비밀번호에 꼭 대문자, 소문자, 특수문자가 들어가야 한다면
    (?=대문자)(?=소문자)(?=특수문자) 와 같은 형태로 사용할 수 있다.

    (?=.*[A-Z])(?=.*[a-z])(?.*[^\w]).*

    로 매칭시키면 된다. 예를들어 aBc123! 이 비밀번호라고 치면

    a. 전방탐색으로 대문자가 일치될 때까지 찾는다. 단, 전방탐색 조건에 .* 이 앞에 붙어있으므로 이전의 모든 문자열도 같이매칭된다.
    대문자 조건 + .* 이 붙어있기 때문에 aB까지 매칭된다. 그러나 전방탐색이어서 이 결과를 날리고 다시 처음부터 탐색한다.

    b. 위와 마찬가지로 소문자

    c. 위와 마찬가지로 특수문자

    d. 결국 모든 전방탐색에 붙은 .* 조건 때문에 마지막 REGEX를 수행할 때 다시 처음부터 매칭된다.
    이에 .* 조건을 하면 비밀번호의 모든 문자열이 매칭되며, 결과가 true로 나온다. 참 쉽죠?
