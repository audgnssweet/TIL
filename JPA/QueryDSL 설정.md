## QueryDSL 설정

```groovy
id "com.ewerk.gradle.plugins.querydsl" version "1.0.10"
```

    Gradle Plugin 설정해준다.

```groovy
implementation 'com.querydsl:querydsl-jpa'
```

    Gradle dependency 추가해준다.

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