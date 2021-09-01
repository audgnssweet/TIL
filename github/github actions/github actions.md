<h2> Github Actions </h2>

    Github Actions?
    소프트웨어 개발 라이프사이클 안에서 PR, PUSH 등의 이벤트 발생시
    원하는 작업을 자동화하여 진행할 수 있게 해 주는 기능

    즉 CI/CD를 자동화해줄 수 있는 툴이다.
    PR을 올렸을 때 테스팅을 하거나 
    Cron을 이용해 특정 시간에 크롤링을 한다던지 하는 행위를 할 수 있다.

---

<h5> 구성 요소 </h5>

    1. workflow
    Github Repository에 설정하는 자동화된 명령이 목록이다. 
    여러 개의 Job으로 구성되어 있으며 특정 이벤트 혹은 특정 시간에 실행되도록 설정 가능하다.
    빌드, 테스트, 배포 등을 자동화 할 수 있으며 .github/workflows 폴더에 YAML 형태로 저장한다.

    2. event
    workflow를 실행시킬 수 있는 특정 행위 (Push, PR, Commit 등)
    
    3. job
    한 Runner에서 실행되는 List<action> 의미
    독립적으로 구성할 수도 있고, 다른 job과 순서상 의존관계를 설정해줄 수 있다.
    (전단계 성공 시 실행 등)

    4. action
    List<Step>의 의미로, 재사용 가능한 workflow의 가장 작은 단위이다.

    5. step
    한 job 안에서의 task를 의미한다. 한 job안에서는 step들끼리 데이터 공유가 가능하다. 

    6. runner
    job을 실행해주는 녀석