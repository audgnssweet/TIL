# 💡 Mockito

Mockito란? 테스트시 Mocking 을 할 수 있도록 도와주는 라이브러리  
(개인적으로 단위테스트는 해당 클래스를 제외한 나머지 모든 의존성은 mocking이 필요하다고 생각한다.)

<br>

## 📜  Mock과 Spy 차이
    
    Mock은 Mock객체를 하나 만드는거고,
    
    Spy는 실제 객체를 Mock library가 한겹 감싸서 (jpa의 collection wrapping처럼) partial mocking 할 때 사용한다. (Mocking 하지 않으면 원래 코드가 동작한다.)
    
<br>

## 📌 mockito
    
    mock 선언방법
    
    1. 단위 테스트시
    Mock, Spy
    2. 통합 테스트시
    MockBean, SpyBean
    
    mocking 방법 (기본적으로 when 사용)
    
    1. when(myService.doSomething(any())).thenReturn(value)
    2. doReturn(value).when(myService).doSomething(any())
    3. doNothing().when(myService).doSomething(any())

<br>
    
## 📌 BDDmockito
    
    
    테스트에는 흐름이있는데,
    
    given, when, then 들어봤을 것이다.
    
    그런데 기존 Mockito에서는 mocking 자체를 when 이라는 키워드를 통해서 한다.
    
    하는 일은 given 에서 하는건데 키워드는 when 이다 → ??
    
    그래서 BDDMockito에서 동작은 똑같은데 given 을 이용해서 할 수 있도록 만든 것.
    
    mock 선언 방법
    
    1. 단위 테스트시
    Mock, Spy
    2. 통합 테스트시
    MockBean, SpyBean
    
    mocking 방법 (given이 주)
    
    1. given(myService.doSomething(any()).willReturn(value)
    2. willReturn(value).given(myService).doSomething(any())
    3. willDoNothing().given(myService).doSomething(any())

<br>
    
## 📌 Mockk (kotlin mock library)
    
    
    Mock 선언 방법
    
    1. 단위 테스트
    mockk, spyk
    2. 통합 테스트
    MockkBean, SpykBean
    
    mocking 방법
    
    1. every { myService.doSomething(any()) } returns value
    2. justRun { myService.doSomething(any()) }
    
<br>

## 🎯 주의점 (이게 핵심)

1. 코틀린에서의 문제점

        자바는 어떤 타입이든 기본적으로 모두 null 을 허용하나, 코틀린은 null이 가능한것과 그렇지 않은 것을 엄격하게 구분한다.
    
        그래서 코틀린에서 mocking 을 할 때, BDDMockito.any() 를 그냥 넣어주게되면
    
        any() 는 기본적으로 null 을 반환하기 때문에 NPE가 발생하는 것이다.
    
        이걸 해결하고자 공식문서를 보면, 분명히 BDDMockito.any(type) 이건 not null 인것을 반환한다고 하는데
    
        그럼에도 불구하고 NPE가 난다. → ???? 머야 ;;
    
        그래서 이걸 해결할 수 있는 방법은
    
    ```kotlin
    protected fun <T> any(type: Class<T>): T {
        return Mockito.any(type)
    }
    ```
    
    이걸 any(type) 을 이렇게 해서 이걸 BDDMockito.any(type) 대신에 사용해주면 잘 작동한다.
    왜인지는 모르겠다;
    
2. 동작 방식의 차이
    
        1. mockito
            1. when().thenReturn()
            2. doReturn().when()
        2. BDDMockito
            1. given().willReturn()
            2. willReturn().given()
    
---

    둘 다에서 나타나는 점인데, a 와 b 의 가장 큰 차이점은 Spy / SpyBean를 사용할 때 나타난다. 
    실제로 코드가 동작이 되냐 안되냐의 차이점이다.
    
    a는 실제로 코드가 동작하나, 그 결과값을 보장하는 것이고
    b는 실제로 코드가 동작하지 않고 결과값을 return 한다.
    
    근데 Mock 이나 MockBean 은 애초에 원래 코드가 없기에 a, b 에서 차이가 없다.
    문제는 Spy나 SpyBean에서이다. 이건 실제로 코드가 돌아가냐 마냐의 차이기때문에 문제가 생긴다.
    
    어떤 상황에서 문제가 생기냐면
    
    위에서 말했듯 any NPE 문제는 any(type) 함수를 만들어줌으로써 해결할 수 있는데
    
    SpyBean 에는 이게 안먹는다; 코드가 실제로 실행돼서 그러는건지
    
    똑같이 NPE가 발생한다. (이거땜에 몇시간 삽질했다)
    
    그래서 kotlin 에서 spyBean을 mocking 할 때 인자가 not null type이라면 (거의 그럴것)
    
    반드시 doReturn().when() 이나 willReturn().given() 을 사용해야 한다.
    
    그래야 코드가 실행이 안되고 바로 return 해서 NPE가 발생하지 않기 때문이다.
    
    아니면 애초에 spyk spykbean 을 사용해서 every 로 mocking 하면 된다 (문제 안생김)
    
<br>

### doNothing, willDoNothing, justRun
    
    반환값이 없는 void method 에서만 가능하다.  
    참고로 반환형이 void 인 메서드와 mock 이라면 mocking 을 하지 않아도 무시하고 넘어가게 된다.
    
<br>

### mockk, spyk, spykBean, mockkBean