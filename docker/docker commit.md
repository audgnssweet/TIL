## Docker commit

    Docker commit 은 컨테이너가 실행된 상태에서, 해당 컨테이너를 source로 새로운 image를 만들기 위한 CLI이다.
    
#### 예제

    Docker에 ubuntu를 올린 직후에, nano 에디터가 설치되어 있지 않다.
    그상대에서 apt update && sudp apt install nano 로 nano를 설치하고, 이를 commit하여
    새로운 이미지로 만든 후에 실행시켜보자.

![image](https://user-images.githubusercontent.com/19279163/134297042-a2c03942-5387-4964-9160-fd6c351afa21.png)
![image](https://user-images.githubusercontent.com/19279163/134297076-7a32ef3c-314b-42ea-b250-0b21dc7480e7.png)


    현재 ubuntu가 올려져있고, nano가 설치되지 않은 상태이다. 여기서 nano를 설치해보자.

![image](https://user-images.githubusercontent.com/19279163/134297432-c9abb8b5-e95a-4b8d-9085-2ae713979953.png)

    위처럼 nano를 설치하고, 이미지로 만든다.

![image](https://user-images.githubusercontent.com/19279163/134297706-b7a21287-b1aa-4cde-8e15-3b7359910b49.png)
![image](https://user-images.githubusercontent.com/19279163/134297831-eec264be-4ddf-4f5a-b2e6-3f8f7cb4c097.png)

    이미지가 잘 생성되었음을 알 수 있다.

![image](https://user-images.githubusercontent.com/19279163/134297868-f11a91b5-70cd-47db-aa40-af5a4c725b99.png)

    이제 이 컨테이너로 접속하여 nano version을 체크하면?
    자동으로 깔려있는 것을 확인할 수 있다.


### 정리

    CLI

    docker commit [이미지이름] [repository이름]:[tag이름]
    
    기능 : 현재 실행중인 container로 새로운 image를 만든다.