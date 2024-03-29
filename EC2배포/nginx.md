# Nginx 배포하기(with.EC2)

## EC2(우분투) 기준 Nginx 배포

### npm 설치

```bash
npm install # npm 설치
npm -v # npm 설치 및 버전 확인
node -v # node 버전 확인
```

### yarn 설치

```bash
yarn install # yarn 설치
yarn --version # yarn 설치 및 버전 확인
```

### npm 및 node 버전

- npm과 node를 처음 설치했을때 최신버전이 아닌 예전 버전이 설치된다.
- npm bulid를 했을때 지속적인 오류가 발생한다면, npm 버전을 6버전 이상 노드를 14버전 이상으로 업그레이드 후 다시 build 시도
- EC2 우분투 내에서 npm 및 node 버전을 업그레이드 하는것은 매우 번거로운 작업이다.
- 쉽게 하기 위해 외부 링크에서 npm 및 node버전 업글을 위한 스크립트가 짜여있는 파일을 받은 후 스크립트를 실행하여, 업그레이드 진행

```bash
curl -sL https://deb.nodesource.com/setup_14.x -o nodesource_14_setup.sh # npm및 node버전 업글을 위한 스크립트 다운
sudo bash nodesource_14_setup.sh # 스크립트 실행
```

### Nginx 설치

```bash
sudo apt install nginx
nginx -v # 설치 및 버전 확인
```

### 프론트 Build 파일 클론

- Nginx에 올릴 프론트 파일을 깃헙에서 가져오자.

```bash
git clone -b {브랜치 이름} --single-branch {리포지토리 주소}
```

### npm build

- 가져온 프론트 파일을 npm으로 build를 해준다.
- build를 하면 build 라이브러리가 생성되는데 이 build 라이브러리 경로를 잘 설정해줘야 문제없이 Nginx를 배포할 수 있다.

```bash
npm start # npm 실행
npm run build # npm을 통한 build
```

### Nginx 설정

```bash
cd /etc/nginx # nginx 설정을 위해 이동
cd sites-abailable # 라이브러리로 이동
vi {파일이름}.conf # 설정을 위한 파일을 vi로 생성과 동시에 열기

server {
        listen 80;
  location / {
    try_files $uri $uri/ /index.html;
    index index.html index.htm;
    root {아까 생성한 build 라이브러리 경로};
  }

  location /api {
      proxy_pass http://localhost:8080;
  }
}

sudo ln -s /etc/nginx/sites-available/{설정 파일 이름}.conf /etc/nginx/sites-enabled # vi 종료 후 심볼릭 링크 디렉토리 생성
vi {설정 파일 이름}.conf # 심볼릭 디렉토리로 이동하여 설정 파일 이름과 동일한 파일을 만들고 수정
ll # {설정 파일 이름}.conf -> /etc/nginx/sites-available/{설정 파일 이름}.conf -> 링크 연결 확인

server {
        listen 80;
  location / {
    try_files $uri $uri/ /index.html;
    index index.html index.htm;
    root {build 라이브러리 경로};
  }

  location /api {
      proxy_pass http://localhost:8080;
  }
}

```

### Nginx 실행

```bash
sudo service nginx configtest # Nginx 설정파일 테스트
* Testing nginx configuration [ OK ] # 라고 나오면 테스트 성공
sudo service nginx start # Nginx 실행
sudo service nginx stop # Nginx 중지
sudo service nginx restart # Nginx 재실행
sudo service nginx reload # Nginx 서버를 종료하지 않고 설정 파일만 다시 적용
```

- 위에 명령어를 상황에 맞게 사용하여 Nginx를 실행하면 된다.

----

## 배포 후기

- 빌드 및 배포에 성공을 하였지만, 정확하게 무엇을 위한 설정이였고 어떤 원리로 작동되는지 아직 다 공부하지 못했다.
- Nginx에 대해 좀 더 공부할 필요가 있을거 같다.