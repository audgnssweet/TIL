<h2> REGEX EXAM </h2>

    패턴매칭은 항상 앞에서부터 된다는 점에 주의
    
---

    1. 비밀번호 패턴매칭 분석

    (?=.*[a-z])(?=.*[A-Z])(?=.*[0-9])(?=.*\W)(?!.*\s).{8,20}

    우선 전체적인 구조부터 분석해보자.
    (?=REGEX_1)(?=REGEX_2)(?=REGEX_3)(?!REGEX_4)REGEX_5
    의 구조로, REGEX 1,2,3을 만족하며 REGEX 4 는 만족하지 않아야만
    비로소 REGEX_5로 매칭시키는 REGEX이다.

    lookaround를 사용했기에 조금 난해할 수 있지만, 분석해보면

    a. (?=.*[a-z])

    이것은 positive lookahead로 REGEX_5를 매칭시키기 전에 먼저
    소문자 1개 이상 있어야하고, 그 앞엔 뭐가 있든 없든 상관없다.
    라는 조건을 주고있다.

    b. (?=.*[A-Z])

    이것도 마찬가지로, 대문자에 해당한다.

    c. (?=.*[0-9])

    이것도 마찬가지로, 숫자에 해당한다.

    d. (?=.*\W)

    이것은 대/소문자/숫자 외에 다른것 (특수문자) 를 요구한다.

    e. (?!.*\s)

    negative lookahead인데, 어떤 공백문자 하나라도 허용하지 않는 것을 요구한다.

    f. .{8,20}

    마지막으로 본 조건인데, 앞의 모든 내용을 통과하면
    그 어떤 문자라도 8-20자 사이로 매칭시키겠다. 라는 것을 의미한다.

---



    