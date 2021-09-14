## QueryDSL 설정

```groovy
id "com.ewerk.gradle.plugins.querydsl" version "1.0.10"
```

    Gradle Plugin 설정해준다.

```groovy
implementation 'com.querydsl:querydsl-jpa'
```

![image](https://user-images.githubusercontent.com/19279163/133209339-1528892b-ae4f-4b3f-a1a0-8cdbb3fe23d9.png)

    Gradle dependency 추가해준다.
    위 사진처럼 library들이 자동으로 추가된다.

    querydsl-core 는 실제로 querydsl을 이용해서 쿼리를 짜는 것을 도와주는 것,
    querydsl-jpa 는 jpa에 특화된 querydsl 기능을 제공한다. (mongodb등 다른것도 있음)

```groovy
def querydslDir = "$buildDir/generated/querydsl"

querydsl {
 jpa = true
 querydslSourcesDir = querydslDir
}

sourceSets {
 main.java.srcDir querydslDir
}

configurations {
 querydsl.extendsFrom compileClasspath
}

compileQuerydsl {
 options.annotationProcessorPath = configurations.querydsl
}
```

    QueryDSL 관련 기타 설정을 해준다.

    이후에 gradle compileQuerydsl 을 돌려주면 Q클래스들이 생성된다. 

    def querydslDir = "$buildDir/generated/querydsl"
    -> build/generated/querydsl 안에 QClass들을 생성하겠다.

    configurations {
        querydsl.extendsFrom compileClasspath
    }
    compileQuerydsl {
        options.annotationProcessorPath = configurations.querydsl
    }
    -> compile classPath에 추가하고, querydsl을 위한 annotation processort을 제공해라.

---

    이런 설정들은 자주 바뀐다.