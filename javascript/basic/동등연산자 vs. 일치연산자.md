## 동등연산자 (==, !=) vs. 일치연산자 (===, !==)

    [결론]
    동등연산자는 값만을 비교하고, 일치연산자는 자료형까지 함께 비교한다.

```javascript
const age = 1;
const age_2 = "1";

//같다고 출력
if (age == age_2) {
  console.log("같다");
}

//출력 X
if (age != age_2) {
  console.log("다르다");
}

//출력 X
if (age === age_2) {
  console.log("같다");
}

//다르다고 출력
if (age !== age_2) {
  console.log("다르다");
}
```

    age는 수, age_2는 문자인데도 동등 연산자로 비교했을 때는 그 값만을 보고 같다고 판단하고
    일치연산자를 사용했을 때는 자료형까지 비교했기 때문에 다르다고 판단했다.
