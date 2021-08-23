<h3> CI CD With Github Actions & AWS Code deploy </h3>

    Github Action으로 Spring boot CI CD 설계하기
    - 삽질 너무 오래해서 내가보려고 만듦

---

    1. 기본적으로 EC2에 배포를 해야하기 때문에, EC2 인스턴스를 만들어준다.
    주의점은, 태그를 꼭 만들어야 한다는 것. (key = Name, value = 원하는 ec2이름)
    인바운드 규칙에 신경써주면 된다.

    2. 위 EC2는 Spring boot application 이 직접 돌아갈 뿐 아니라, AWS CodeDeploy APP을 통해
    배포가 이루어질 인스턴스이기 때문에, IAM 역할을 지정해줘야 한다.
    IAM 역할은 EC2가 어떤 역할을 할지 지정 (AWS 서비스 지정 ec2)이 가능하다.
    -> 역할은 AmazonEC2FullAccess, AmazonS3FullAccess
    
    3. EC2 인스턴스를 중지시켜서 보안 -> IAM 역할 수정에서 위에서 만든 EC2의 IAM 역할을 지정해준다.
    그리고 인스턴스 재부팅

    4. S3 버킷을 디폴트 설정으로 만들어준다.

    5. IAM 에서 사용자 그룹을 생성해준다. 이는 IAM 사용자에게 적용하기 위한 그룹이다.
    AmazonEC2FullAccess
    AWSCodeDeployFullAccess
    AWSCodeDeployRole
    AmazonS3FullAccess

    6. IAM 사용자를 생성해준다. 그리고 바로 위에서 만든 사용자 그룹에 속하게 설정해준다.
    반드시 Access Key와 Private Access Key를 저장해줘야 한다.

    7. CodeDeploy용 역할을 생성해준다. 어차피 권한이 한개여서 따로 설정 안해줘도 된다.
    
    8. CodeDeploy 애플리케이션을 만들어준다. 그리고 CodeDeploy 그룹을 만들어준다.
    그룹을 만들 때 서비스 권한을 바로 위에서 만든 CodeDeploy 역할로 지정해주면 된다.

    여기까지 하면 빌드 -> 압축 -> S3에 업로드 까지 준비가 완료되었다.

    9. 이제 CodeDeploy용 script를 작성할 차례이다. repository root에 scripts 폴더를 만들어준다.
    
    10. before-install.sh

    rm -rf /home/EC2_OS(ubuntu)/PROJECT_NAME

    여기서 EC2_OS는 인스턴스를 생성할 때 지정했던 OS로 해주면 된다.
    보통 Ubuntu (ubuntu) 혹은 Amazon Linux (ec2-user) 로 설정한다.
    PROJECT_NAME은 프로젝트 이름을 임의로 설정. 반드시 프로젝트 root폴더이름과 같거나 할 필요 없다.

    11. after-install.sh

    cd /home/EC2_OS/PROJECT_NAME
    sudo chown -R EC2_OS:EC2_OS /home/EC2_OS/PROJECT_NAME
    chmod 777 /home/EC2_OS/PROJECT_NAME
    chmod 777 /home/EC2_OS/PROJECT_NAME/*/**
    
    EC2에서 기본적으로 제공하는 사용자는 Amazon Linux라면 ec2-user, Ubuntu라면 ubuntu이다.
    그 외에는 접속이 안되니, CodeDeploy가 이 사용자의 힘을 빌려 배포를 할 수 있도록 권한을 변경해줘야 한다.

    12. application-start.sh

    sudo pkill -6 java
    nohup java -jar /home/ubuntu/corona-backend-dev/build/libs/*.jar > /dev/null 2>&1 &

    spring boot는 java application이기 때문에 별다른 설정이 없다면 java라는 이름의 process로 구동되고 있다.
    새로운 배포를 하기 전에 먼저 프로세스를 죽이고, 새로 실행한다.
    중요한점은 .jar뒤의 부분인데, Linux에서는 기본적으로 SSH를 통해 스크립트를 수행시, 표준 출력이 닫히거나
    timeout이 발생할 때 까지 스크립트가 계속해서 열려있게 된다. 이때문에 CodeDeploy상으로
    배포 진행중이 끝나지 않게 되는데, 이를 해결하기 위해 출력을 돌리는 작업이다.

    13. 실제로 CodeDeploy가 읽을 수 있도록 repository root에 appspec.yml파일을 만들어준다.
    
    version: 0.0
    os: linux

    files:
        - source: /
        destination: /home/EC2_OS/PROJECT_NAME
    permissions:
        - object: /home/EC2_OS/PROJECT_NAME
        owner: EC2_OS
        - object: /home/EC2_OS/PROJECT_NAME/*/**
        owner: EC2_OS
    hooks:
        BeforeInstall:
            - location: scripts/before-install.sh
        AfterInstall:
            - location: scripts/after-install.sh
        ApplicationStart:
            - location: scripts/application-start.sh

    어떤 시점에 어떤 script를 실행할지 정의해주는 yml파일이다.
    
    여기까지 하면 거의 모든 준비가 끝났다고 해도 무방하다.
    이제 EC2를 설정해보자

    14. EC2 접속 후 Update Upgrade OpenJdk 설치

    sudo apt-get update && sudo apt-get upgrade
    sudo apt-get install openjdk-11-jdk

    15. CodeDeploy가 ruby 기반인데, 2.4 버전 이후로 동작하지 않는 이슈가 있어 버전을 맞춰준다.

    sudo apt remove --purge ruby
    sudo apt autoremove
    sudo apt -y install software-properties-common
    sudo apt-add-repository ppa:brightbox/ruby-ng
    sudo apt update
    sudo apt install -y ruby2.4

    ruby --version으로 확인했을 때 2.4가 나오면 성공이다.

    16. 이제 CodeDeploy를 설치해주자
    
    sudo apt install wget
    cd
    wget https://aws-codedeploy-ap-northeast-2.s3.ap-northeast-2.amazonaws.com/latest/install
    chmod +x ./install
    sudo ./install auto

    잘 설치되었는지 
    sudo service codedeploy-agent status
    로 확인해주고, 만약 detected both ... 에러가 생긴다면
    sudo ./install deb
    로 해결해준다.

    17. 환경변수를 등록해주자
    CodeDeploy는 일반적인 환경변수 ex) .bashrc 등을 읽지 못하기 때문에, sh 스크립트를 통해 사용하는 방법(일반적)
    이 있다. 그러나 나는 CodeDeploy가 인식할 수 있는 환경변수 경로인
    /etc/.profile.d/codedeploy.sh
    에 export로 환경변수를 등록해주는 방법을 채택했다.
    여기까지 등록했으면
    sudo service codedeploy-agent restart 
    를 통해 CodeDeploy를 재시작해준다.
    참고로, 
    sudo service codedeploy-agent status
    로 상태를 확인할 수 있다.

    18. 마지막으로 이 CI CD 과정은 Github Action으로 작동하는 것이기에
    Github Action을 위한 Script를 workflow에 작성한다.
    이 Script는 너무 길어서 CD yml로 따로 첨부한다.

    19. 여기까지 성공했다면 끝이다. 이제 develop, master에 code가 merge되면 자동으로 배포가 될 것이다.
    