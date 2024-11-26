# EC2 인스턴스에 Docker로 nextjs 프로젝트 올리기

## 들어가며
사이드 프로젝트를 시작하기 앞서, CI/CD를 구축하면서 학습한 내용을 정리해본다.
(EC2 인스턴스 설정, 도커 설치는 생략)

- Docker 환경에서 nginx와 nextjs 이미지를 구동해보기
- nginx 80포트를 접속하면 3000포트로 리다이렉트


### Docker에서 nginx와 nextjs 이미지 구동시키기

1. nextjs 프로젝트 상단에 `Dockerfile` 작성하기
> Dockerfile은 Docker Image를 생성하기 위한 스크립트임

처음에는 아래와 같이 간단하게 작성했다.
```shell
FROM node:18-alpine AS base # node v18을 베이스 이미지로 사용

WORKDIR /usr/src/app # 작업 디렉토리 설정

COPY package.json ./ # yarn install을 위해 package.json 복사

RUN yarn # Dependency 설치

COPY . . # 전체 디렉토리 복사

RUN yarn build # 빌드 실행

EXPOSE 3000 # 3000포트 열기

CMD ["yarn", "start"] # 실행
```
참고로, 빌드를 `standalone` 옵션으로 수행한다면 실행 명령어를 `node ./next/standalone/server.js`로 수정해야 한다.   
(나는 `package.json` 파일에서 `start` 명령어를 직접 수정해줬다.)

여기까지가 nextjs 프로젝트 빌드 이미지를 생성하기 위함이다.
다음으로는 `ngnix` 이미지를 생성해준다.
두 개의 이상의 이미지를 도커 환경에서 실행시키려면 `docker-compose`를 사용해야한다.

2. `docker-compose.yml`로 nginx 이미지 생성과 nextjs 빌드 이미지 실행하기

```shell
version: '3'

services:
  nextjs:
    container_name: sharenode-front # 이미지 실행하면 생성되는 컨테이너 이름 설정
    build: 
      context: ./ # 현재 프로젝트 위치 명시
      dockerfile: ./Dockerfile # 현재 프로젝트 내 Dockerfile 위치 명시
    restart: always # always 컨테이너가 멈추면 항상 다시시작 함, 수동으로 중지된 경우 재시작되거나 컨테이너 자체가 수동으로 재시작될때만 재시작됨
    ports:
      - "3000:3000"

  nginx:
    image: nginx:latest # 도커 허브에서 이미지 가져옴
    container_name: web_server
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d:rw
    depends_on: # 위 nextjs 작업이 끝나면 실행한다는 의미
      - nextjs

networks:
  my_network: # 두 개의 다른 컨테이너를 하나의 네트워크로 관리하여 통신함 
    external: true
```

3. 도커 실행하기
`docker compose up -d`   
`-d` 옵션은 백그라운드에서 실행하겠다는 의미이다.

위 명령어를 치고나서 ec2에 존재하는 퍼블릭 IP로 접근(포트 80) 하면 nginx 실행 화면을 볼 수 있다.

4. 80포트 3000포트로 포워딩하기
