# Docker 기본 명령어

&nbsp;

# 1. 들어가며

도커(Docker)는 요새 개발할 때 이제 거의 필수가 되어가고 있는 것 같습니다. 애플리케이션을 만들고 배포할 때 이제는 도커 이미지로 만들어서 컨테이너화 시키는 것이 보편화되고 있습니다. 인턴 생활을 하며 도커 이미지를 만들어서 AWS ECS 와 Github Action 을 이용해서 CI/CD 파이프라인을 구축해본 적이 있는데요. 그 때는 자세히 알고 했다기보단 구글링을 통해 얼추 따라했던 것 같습니다.

백엔드 개발자로서 도커는 이제 거의 필수지식이 되어가는만큼 이번 기회에 도커에 대해서 공부를 해보고자 합니다. 이번 글에서는 도커에 대한 개념보다는 도커에 관련된 명령어들에 대해서 정리해보고자 합니다. 도커 개념 및 컨테이너의 개념에 관해서는 차후 다른 글에서 한 번 다뤄보도록 하겠습니다.

&nbsp;

# 2. 자주 쓰이는 Docker 명령어

> docker run {이미지 이름}

도커 컨테이너를 실행시키는 명령어입니다.

> docker run -d {이미지 이름 }

도커 컨테이너를 백그라운드에서 실행시킵니다. (컨테이너를 종료하지 않아도 바로 명령 프롬프트창으로 돌아와서 명령어를 입력할 수 있다.)

> docker ps

실행 중인 컨테이너 목록과 컨테이너의 정보들을 출력합니다.

> docker ps -a

실행 중인 컨테이너 뿐 아니라 정지되어 있는 컨테이너까지 모두 출력합니다.

> docker stop {컨테이너이름}

실행 중인 컨테이너를 멈추게 합니다.

> docker rm {컨테이너 이름}

컨테이너를 영구적으로 삭제합니다.

> docker {images}

도커 이미지들을 불러옵니다.

> docker rmi {이미지 이름}

도커 이미지를 삭제합니다. 도커 이미지를 삭제하기 위해서는 이미지로 실행된 컨테이너가 중단되거나 삭제돼 있어야 합니다.

> docker pull {이미지 이름}

도커 이미지를 다운로드 합니다. 

> docker exec {컨테이너 이름 or 컨테이너 아이디} {수행할 명령어}

컨테이너에게 명령어에 해당하는 동작을 시킵니다.

> docker attach {컨테이너 ID 또는 컨테이너 이름}

백그라운드에서 실행 중인 컨테이너를 다시 연결 모드로 바꾸고 싶을 때 사용합니다. 

&nbsp;

# 3. 마치며

위에서 정리한 명령어들은 도커를 사용하며 가장 기본적으로 자주 사용하는 명령어들입니다. 이 명령어들을 기반으로 다양한 옵션을 주면서 명령어를 더 실행할 수 있는데요. 세부적인 내용에 대해서는 별도로 카테고리별로 한 번 더 정리해보도록 하겠습니다.q