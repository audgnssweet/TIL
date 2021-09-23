## undifined vs. null

    결론부터 말하면

    [공통점]
    자료형임과 동시에 값이다

    [차이점]
    undifined는 선언만하고 정의하지 않았을 때,
    null은 명시적으로 값이 비어있음을 나타내고 싶을 때 사용한다.

---

### undifined

```javascript
var _var;
console.log(_var);

let _let;
console.log(_let);

const _const = undefined; // 반드시 선언과 동시에 초기화 해야하기 때문
console.log(_const);
```

    위처럼 어떤 키워드로든 변수를 선언만 해주면 undifined 라는 값이 들어가있다.
    말 그대로 선언만 되고 아직 정의되지 않은 상태를 의미한다.

```javascript
//아래는 전부다 같다고 나온다.
if (undefined == undefined) {
  console.log("같다");
}

if (undefined === undefined) {
  console.log("같다");
}
```

    둘은 모두 같다고 출력된다.
    왜냐하면 undifined의 자료형은 undifined이고, 값도 undifined이기 때문이다.

---

### null

```javascript
var _var = null;
console.log(_var);

let _let = null;
console.log(_let);

const _const = null;
console.log(_const);
```

    null은 undifined와 달리 무조건 null로 명시적 초기화가 필요하다.
    왜냐하면 명시적으로 값이 비어있음을 알리기 위함이기 때문이다.

```javascript
if (null == null) {
  console.log("같다");
}

if (null === null) {
  console.log("같다");
}
```

    둘은 모두 같다고 출력된다.
    왜냐하면 null의 자료형은 null 이고, 값도 null 이기 때문이다.

---

## 비교

```javascript
//같다고 출력
if (null == undefined) {
  console.log("같다");
}
//출력 X
if (null === undefined) {
  console.log("같다");
}
```

    null과 undifined를 동등연산자로 비교하면 같다고 출력하고,
    일치연산자로 비교하면 다르다고 출력한다.

    undifined와 null은 그 사용의 용도가 다른만큼 javascript에서는 일치연산자를 이용해 비교하는게 맞는 방법이다.
