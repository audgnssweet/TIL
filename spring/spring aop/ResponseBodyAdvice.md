<h3> ResponseBodyAdvice </h3>

    언제 사용하는가?
    AOP를 통해 Response를 조작하고 싶을 때 사용한다. (Body든 Header든)

    나의 경우에는 모든 API 헤더에 응답시 공통으로 적용해야하는 로직이 있어 사용했다.

    Spring AOP의 Interceptor과 다른 점은, Interceptor는 직접 Response Body에 접근할 수 없는 반면,
    ResponseBodyAdvice를 사용하면 직접 Response Body에도 접근할 수 있다.

---

<h5> 사용해보자 </h5>

    먼저 사용법이다
    1. @ControllerAdvice 어노테이션을 붙여준다.
    2. ReponseBodyAdvice 인터페이스를 상속받는다. (제네릭임에 주의)
    3. override한 supports method에 원하는 controller 조건이 true가 되도록 만든다.
    4. override한 beforeBodyWrite method에 body나 header를 조작한다.

---

<h5> 코드를 보자 </h5>

```kotlin

    @ControllerAdvice
    class CustomResponseBodyAdvice<T> : ResponseBodyAdvice<T> {

        override fun supports(returnType: MethodParameter, converterType: Class<out HttpMessageConverter<*>>): Boolean {
            return returnType.containingClass == CustomController::class.java
        }

        override fun beforeBodyWrite(
            body: T?,
            returnType: MethodParameter,
            selectedContentType: MediaType,
            selectedConverterType: Class<out HttpMessageConverter<*>>,
            request: ServerHttpRequest,
            response: ServerHttpResponse
        ): T? {
            response.headers["custom AOP header"] = "custom AOP Header"
            return body
        }
    }

```

    1. returnType.containingClass를 통해 어떤 Controller에서 method가 반환되는지 판별 가능하다.
    2. response의 headers나 body 자체를 원하는대로 조작해준다.