# RaspiCam
A Project with raspberry pi camera, web and iOS

# 20201021
- 클라우드 서비스(네이버클라우드)에서 대여한 서버에 해당 origin remote의 issue3 브랜치 받아오기
- Ubuntu 16.04 위에 Python 개발 환경 설정 및 Django 환경 구축
- Python: 3.9
- Django: 2.2.6
- 기타 설치: djangorestframework, markdown, django-filter
- 포트포워딩 통해서 외부접속 확인 완료

# 20201023
- images 앱 생성
- model 생성
- serializer 생성
- views에 이미지 수신하는 함수 생성
- url에 맵핑

# 20201031
- images에 html 파일을 넣을 templates 폴더 생성(index.html 생성 후 static 렌더링 설정)
- images에 css, js, images 등 리소스를 넣을 static 폴더 생성(추후 유환님이 넣을 수 있게)
- settings.py 에 static 관련 사전 설정
- url에 맵핑
- SERVER라는 상위폴더 생성후 서버 관련 파일 전체 이동

# 20201107 ~ 20201112
- Dockerfile 생성 및 작성
- docker-compost 생성 및 작성
- alpine linux 이미지 위에 python, django 등 환경 설정 완료
- Pillow를 제대로 받아오지 못하는 오류 발생
- importError: Error loading shared libraries libjpeg.so.8:
- 다양한 자료를 참고하여 패키지들을 add 해주었는데 타 리눅스 image 기반으로 작성되어 있어서 그런지 non-zero code 에러 연달아 발생
- https://github.com/python-pillow/Pillow/issues/1763
- https://stackoverflow.com/questions/56407216/django-does-not-find-pillow-though-installed
- 위 링크 들을 참고
- https://pkgs.alpinelinux.org/packages 공식 문서에서 패키지 검색하며 찾음
- docker image를 rebuild 하는 법을 몰라서 docker rmi -f 로 이미지 삭제 후 진행 반복..
- 성공

# 20201113 ~ 20201115
- djongo 설치 후 mongodb와 연동
- mongodb에 데이터가 잘 적재되는지 확인(mongodb compass 활용)
- 실제 이미지 자체는 특정 폴더에 저장하며 db에는 'caption: 해당 이미지 경로', 'created: 생성 시간' 2개의 컬럼 저장
- index.html과 해당 데이터를 렌더링 하여 출력
- static, media 경로 등에 대해 제대로 숙지가 안되어 우선적으로 static > media 폴더 생성 후 이미지를 해당 경로로 저장
- index.html에서 출력시 해당 caption을 slicing하여 {% static image.caption %} 과 같은 식으로 출력함
- 성공

# 20201115 ~ 20201116
- 로컬에서 구현한 django + mongo 부분을 docker-compose로 구현
- mongo image 설치까지는 되나 pymongo.errors.ServerSelectionTimeoutError: mongodb:27017: [Errno -2] Name does not resolve 에러 발생
- stackoverflow, google 등 검색을 해 보았지만 해결 방법을 모색하지 못함
- requirements.txt에 djongo만 명시했으나 pymongo 및 타 패키지들이 같이 설치되어 발생하는 오류로 예상
- 해당 건 자료를 찾아보던 중 stackoverflow에 질문 올림 (https://stackoverflow.com/questions/64854380/docker-pip-install-odd-with-requirements-txt)
- 해당 사항은 오류가 나는 것 아니라면 문제가 없고 자동적으로 종속성에 따라 타 패키지들이 설치된다고 함(예. scipy 설치시 numpy 설치 되는 것 처럼)
- djongo 자체가 돌아가려면 pymongo 패키지가 필요한 것으로 확인됨
- 해당 문제에 대해 2번째 질문 (https://stackoverflow.com/questions/64855098/docker-compose-with-django-mongodb-using-djongo-error-pymongo-errors-serverse)
- mongo 컨테이너 이름을 mongodb로 설정안해줘서 발생했던 문제임을 찾게 됨
- docker-compose.yml 수정('mongo' -> 'mongodb')
- https://github.com/docker-library/mongo/issues/323 해당 문제 발생 확인
- mkdir -p ./mongo/home/mongodb 
- docker-compose.yml 수정(volumes: - ./mongo/home/mongodb:/home/mongodb 추가)
- 성공

# 20201117
- 서버로 git pull
- 서버에 docker, docker-compose 설치
- docker-compose up 진행 중 에러 발생 -> "ERROR: Version in "./docker-compose.yml" is unsupported"
- https://github.com/bigbluebutton/greenlight/issues/228 참고
- $ sudo apt-get remove docker-compose
- $ sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
- $ sudo chmod +x /usr/local/bin/docker-compose
- $ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
- mongo/home/mongodb 부분도 volume 처리가 안되는 것을 발견
- chown -R 999:999 ./home/mongodb https://github.com/docker-library/mongo/issues/323 참고
- git remote에 push할때 빈 폴더들(특히 static/media)이 같이 git으로 관리 안 된 것으로 확인
- 해당 부분 폴더 안에 .gitkeep 파일을 만들어줌으로써 해결
- aws에 ubuntu 18.04 서버 대여 후 테스트
- 이미지 수신 및 DB 저장 확인
- 성공

# 20201118
- 추가 html 파일 렌더링
- url 맵핑: gallery.html -> galleries/, generic.html -> generics/
- 하드코딩 url 맵핑시 문제점 발생 3개 html 파일 기준 예) http://127.0.0.1:8000/galleries/ 여기 위치에서 다른 거 클릭시 중복 돼서 galleries/generics/ 이렇게 요청 됨
- https://wayhome25.github.io/django/2017/03/06/django-06-poll-project-08-url/ 참고하여 작성
- static resource들 처리하여 렌더링 완료
- image animation 부분 처리 방법: 현재까지 서버에 저장된 이미지를 한데 모아 gif 화 시킨 후 출력하기로 결정
- 1. 새로운 이미지가 들어온다
- 2. 기존 gif 이미지를 지운다
- 3. 새로운 이미지와 기존 이미지를 합쳐서 gif 만든다
- 4. 출력해준다
- 테스트 파이썬 파일에서 테스트 완료

# 20201119
- Raspicam 본 서버에 작성 및 테스트 완료