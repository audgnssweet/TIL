<h3> trouble shooting </h3>

    1.
    The deployment failed because no instances were found for your deployment group. 
    Check your deployment group settings to make sure the tags for your Amazon EC2 instances
    or Auto Scaling groups correctly identify the instances you want to deploy to, and then try again.

    CodeDeploy에서 위와같은 에러가 뜬 적이 있다.
    이런 경우에는 CodeDeploy에서 배포 그룹에 들어가, 인스턴스를 다시 선택하고
    1개의 인스턴스가 선택됨. 이라는 표시까지 재확인해주고 설정해주면
    제대로 동작하는 것을 확인할 수 있다.