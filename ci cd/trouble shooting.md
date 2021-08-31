<h3> trouble shooting </h3>

    1.
    The deployment failed because no instances were found for your deployment group. 
    Check your deployment group settings to make sure the tags for your Amazon EC2 instances
    or Auto Scaling groups correctly identify the instances you want to deploy to, and then try again.

    CodeDeploy에서 위와같은 에러가 뜬 적이 있다.
    이런 경우에는 CodeDeploy에서 배포 그룹에 들어가, 인스턴스를 다시 선택하고
    1개의 인스턴스가 선택됨. 이라는 표시까지 재확인해주고 설정해주면
    제대로 동작하는 것을 확인할 수 있다.

    2.
    내가 배포한 애플리케이션은 서버 시간이 중요했는데, 배포한 아마존 서버가 미국에 있어서
    LocalDateTime.now() 를 했을 때 미국 기준 시간으로 실행이 되어서 애를 먹었던 경험이 있다.
    이럴때는 LocalDateTime 말고 ZoneDateTime을 사용해줘야 한다.

    서울의 시간은
    ZonedDateTime.now(ZoneId.of("Asia/Seoul")).toLocalDate()
    처럼 해주면 된다.
