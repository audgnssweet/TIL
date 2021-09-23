## var, let, const

    javascript에는 변수 및 상수를 선언할 수 있는 방식이 var, let, const로 세 가지가 있다.
    ES5 이전까지 가장많이 사용되었던 방식이 var이고, var의 문제점을 극복하고자 let과 const가 등장했다.

    결론부터 말하면 var는 사용을 자제해야하고, let과 const를 사용해야 한다.
    이유는 아래와 같다.
    1. 동일한 이름의 변수를 다시 선언 가능하다
    2. 일반적인 코드규칙인 블록(block) 규칙을 무시한다. (단 함수에서는 적용)
    3. 호이스팅 문제가 발생한다.

    우선 특징을 알아보자

### var

    var은 변수 선언 방식으로 mutable한 변수를 선언할 때 사용한다.

```javascript
var var1 = "jeong";
console.log(var1);

var1 = "jeong2";
console.log(var1);

var var1 = "jeong3";
console.log(var1);
```

    변수의 값을 재할당 할 수 있을 뿐 아니라 똑같은 이름의 변수를 var로 한번 더 선언해도 에러가 발생하지 않는다.

### let

    let은 변수 선언 방식으로 var처럼 변수값을 변경할 수 있으나 같은 이름의 변수를 한번 더 선언하는 것은 용납하지 않는다.

```javascript
let var3 = "jeong";
console.log(var3);

var3 = "jeong2";
console.log(var3);

let var3 = "jeong2"; //에러 발생!
```

### const

    const는 상수 (immutable variable) 을 선언할 때 사용하는 키워드로, 반드시 선언과 동시에 초기화해야하고, 이후에 재할당이 불가능하다.

```javascript
const var2 = "jeong";
console.log(var2);

var2 = "ok"; //에러 발생!
```

---

## var 사용을 자제해야 하는 이유

    1. 동일한 이름의 변수를 다시 선언 가능하다
    2. 일반적인 코드규칙인 블록(block) 규칙을 무시한다. (단 함수에서는 적용)
    3. 호이스팅 문제가 발생한다.

    이중 1번은 위에서 확인했으니 생략한다.

### 2. 일반적인 코드규칙인 블록규칙을 무시한다. (단 함수에서는 적용)

```javascript
function plusb() {
  var b = 0;
  b++;
  console.log(b);
}

plusb();
console.log(b); // 에러 발생!
```

    일반적으로 위와 같은 코드를 작성하면 에러가 난다 왜냐하면 b는 블록 -> {} 안에서 생성된 변수이기 때문에
    생명주기가 블록과 같기 때문이다.

    여기까지만 보면 문제가 없어보인다. (함수에서는 적용)

```javascript
{
  var c = 999;
}

console.log(c);
```

    위와 같은 코드가 문제없이 작동한다.
    이상하지않은가? 하지만 let과 const로 위와 동일한 코드를 작성하면, 에러가 생긴다.

### 3. 호이스팅 문제

    호이스팅.md 를 참고하자
