<h2> REGEX lookaround </h2>

    lookaround 는 regex의 고급개념이다

    positive는 일치, negative는 비일치

    lookahead는 먼저검사, lookbehind는 나중검사 라고 생각하면 된다.

---

    1. Positive lookahead (?=)
    
    (?=REGEX_1)REGEX_2
    우선 주어진 문자열을 REGEX_1과 매칭하는지 먼저 검사한다.
    그리고 REGEX_1과 일치 한다면, 일치되었던 결과를 버리고
    (원래는 앞에서부터 일치시켜 나가기 때문에)
    다시 처음부터 REGEX_2와 일치하는 결과를 매칭시킨다.

    (?=REGEX_1)(?=REGEX_2)REGEX_3
    처럼 여러개의 조건을 선행시키는 것도 가능하다.

    2. Negative lookahead (?!)

    (?!REGEX_1)REGEX_2
    우선 주어진 문자열을 REGEX_1과 일치하지 않는지 먼저 검사한다.
    그리고 REGEX_1과 일치하지 않는다면, REGEX_2를 적용하여 매칭한다.

    마찬가지로 여러개 조건을 주는 것이 가능하다.