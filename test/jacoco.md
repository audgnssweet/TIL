jacoco
=

테스트 coverage 를 측정하기 위한 library

적용해보자

---

```Groovy
//build.gradle

plugins {
    id 'jacoco'
}
```

plugin 을 설정해주면

1. jacocoTestCoverageVerification  
2. jacocoTestReport

두 개의 task 가 생긴다. 

여기서 jacocoTestReport 는 테스트를 기반으로 coverage 를 측정하여 문서화 시켜주는 것이고,  
jacocoTestCoverageVerification 은 지정한 limit 을 넘지 못할 경우 build 를 실패하도록 하는 것이다.

그래서 두 task 는 반드시 test 가 선행된 후에 실행되어야 한다.

> test -> jacocoTestReport -> jacocoTestCoverageVerification

---

여기까지 완료했다면 기타 설정을 할 수 있다.

```Groovy
//build.gradle

jacoco {
    toolVersion = jacoco version
    reportsDir = jacoco report 가 어디에 저장될지 설정 가능하나, jacocoTestReport 에서 설정 가능
}
```

---

여기까지 완료했다면 testReport 관련 설정을 해줄 수 있다.

```Groovy
//build.gradle

jacocoTestReport {
    //원하는 리포트 type을 설정하고, 각각의 디렉토리를 따로 설정 가능하다.
    reports {
        html.enabled(true)
        xml.enabled(false)
        csv.enabled(false)

        //리포트에 포함되지 않도록 패키지 레벨로 설정한다.
        afterEvaluate {
            classDirectories.setFrom(files(classDirectories.files.collect {
            fileTree(dir: it,
                    exclude: [
                            '**/dto/**',
                            '**/error/**',
                            '**/config/**'
                    ] + Qdomains)
            }))
        }

//        html.outputLocation = "$buildDir/jacocoHtml"
    }
}

```

---

여기까지 완료했다면 coverage limit 을 설정 가능하다.

```Groovy
jacocoTestCoverageVerification {

    //coverage rule 들을 정의할 수 있는 곳
    violationRules {

        //여러 개의 rule 을 정의할 수 있는데, 안에 element 가 없으면 total 이다. -> 전체 파일 합친 기준
        rule {
            //제약사항으로, (counter, value, minimum) 을 정의할 수 있다.
            limit {
                // 'counter'를 지정하지 않으면 default는 'INSTRUCTION'
                // 'value'를 지정하지 않으면 default는 'COVEREDRATIO'
                minimum = 0.30
            }

            //경로에는 와일드카드인 * 와 ? 를 사용 가능하다.
            //패키지 + 클래스명 으로 QClass 를 제외해준다.
            includes = [] //rule 적용 대상을 package 수준으로 정의.
            excludes = [] //rule 미적용 대상을 클래스 네이밍으로 정의한다.
        }

        rule {
            //enable disable 설정 가능
            enabled = false
            element = 'CLASS'

            limit {
                //브랜치 커버리지
                counter = 'BRANCH'
                value = 'COVEREDRATIO'
                minimum = 0.90
            }

            limit {
                //라인 커버리지
                counter = 'LINE'
                value = 'COVEREDRATIO'
                minimum = 0.80
            }

            limit {
                //빈줄 제외 코드 라인수를 200 라인으로 제한
                counter = 'LINE'
                value = 'TOTALCOUNT'
                maximum = 200
            }
        }
    }
}
```

---

element
=
커버리지의 체크 기준이다.  

> BUNDLE(패키지 bundle로 default)  
PACKAGE  
CLASS  
SOURCEFILE  
METHOD

counter
=
커버리지 측정의 최소 단위 (java byte code 실행여부)

> LINE (빈 줄 제외 실제라인 수)  
BRANCH(분기) - if, switch  
CLASS - 클래스 내부 메서드가 한번이라도 실행되면 실행  
METHOD - 메서드가 한번이라도 실행되면 실행  
LINE - 한 라인이라도 실행되었다면 실행 (소스 코드 포맷에 영향 받음)  
INSTRUCTION - 바이트코드 실행되었다면 실행 (소스 코드 포맷에 영향 받지 않음)  
COMPLEXITY(복잡도)

value
=
측정한 counter 의 정보를 어떤식으로 보여줄지 결정

> TOTALCOUNT(전체 개수)  
MISSEDCOUNT(커버되지 않은 개수)  
COVEREDCOUNT(커버된 개수)  
MISSEDRATIO(커버안된 비율)  
COVEREDRATIO(커버된 비율)


---

최종 build.gradle

```Groovy
buildscript {
    ext {
        queryDslVersion = "4.4.0"
    }
}

plugins {
    id 'org.springframework.boot' version '2.5.8'
    id 'io.spring.dependency-management' version '1.0.11.RELEASE'
    id 'java'
    id 'jacoco'
}

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
}

test {
    useJUnitPlatform()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-validation'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    compileOnly 'org.projectlombok:lombok'
    runtimeOnly 'mysql:mysql-connector-java'
    annotationProcessor 'org.projectlombok:lombok'


    // QueryDSL
    implementation "com.querydsl:querydsl-jpa:${queryDslVersion}"
    annotationProcessor(
            "javax.persistence:javax.persistence-api",
            "javax.annotation:javax.annotation-api",
            "com.querydsl:querydsl-apt:${queryDslVersion}:jpa")

    // test
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testImplementation 'commons-io:commons-io:2.11.0'

    // h2
    runtimeOnly 'com.h2database:h2'
}

sourceSets {
    main {
        java {
            srcDirs = ["$projectDir/src/main/java", "$projectDir/build/generated"]
        }
    }
}

jacocoTestReport {
    dependsOn test
    reports {
        html.enabled(true)
        xml.enabled(false)
        csv.enabled(false)
    }

    def Qdomains = []
    for (qPattern in "**/QA".."**/QZ") {
        Qdomains.add(qPattern + "*")
    }

    afterEvaluate {
        classDirectories.setFrom(files(classDirectories.files.collect {
            fileTree(dir: it,
                    exclude: [
                            '**/dto/**',
                            '**/error/**',
                            '**/config/**'
                    ] + Qdomains)
        }))
    }
}

jacocoTestCoverageVerification {
    def Qdomains = []
    for (qPattern in "*.QA".."*.QZ") {  // qPattern = "*.QA","*.QB","*.QC", ... "*.QZ"
        Qdomains.add(qPattern + "*")
    }

    violationRules {
        rule {
            enabled = true
            element = 'CLASS'

            limit {
                counter = 'LINE'
                value = 'COVEREDRATIO'
                minimum = 0.8
            }

            limit {
                counter = 'BRANCH'
                value = 'COVEREDRATIO'
                minimum = 1.00
            }

            excludes = [
                    '*.*Dto',
                    '*.*Config',
                    '*.*Exception',
                    '*.*Application'
            ] + Qdomains
        }
    }
}

//gradle task 를 새로 정의하여 test 와는 분리한다.
task testCoverage(type: Test) {
    group 'verification'
    description 'Runs the unit tests with coverage'

    dependsOn(':test',
            ':jacocoTestReport',
            ':jacocoTestCoverageVerification')

    tasks['jacocoTestReport'].mustRunAfter(tasks['test'])
    tasks['jacocoTestCoverageVerification'].mustRunAfter(tasks['jacocoTestReport'])
}
```