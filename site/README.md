# Docker 온보딩 실습

## 프로젝트 소개
nginx 도커 컨테이너로 간단한 HTML 웹페이지를 배포하는 온보딩 실습입니다.

## 실행방법
```bash
docker build -t mysite .
docker run -d -p 8080:80 mysite
```


브라우저에서 http://localhost:8080으로 접속하세요.


## 실행 환경
- OS: Windows (Git Bash / UCRT64)
- Shell: bash
- Docker: Docker version 29.9.2, build a82e528
- Git: git version 2.56.0.windows.4

## 수행 체크리스트
- [x] 터미널 기본 조작 및 폴더 구성
- [x] Docker 설치/점검
- [x] Dockerfile 빌드/실행
- [x] 포트 매핑 접속
- [x] 한글 인코딩 해결
- [x] 바인드 마운트 반영
- [x] 볼륨 영속성
- [x] Git 설정 + VSCode GitHub 연동

## 볼륨 영속성 검증
```bash
# 1) 데이터 쓴 컨테이너 삭제
$ docker rm -f vol-write
vol-write

# 2) 같은 볼륨으로 새 컨테이너 생성
$ MSYS_NO_PATHCONV=1 docker run -d --name vol-read -v mydata:/usr/share/nginx/html nginx

# 3) 새 컨테이너에서 데이터 확인
$ MSYS_NO_PATHCONV=1 docker exec vol-read cat /usr/share/nginx/html/test.txt
볼륨 영속성 테스트 성공
```
→ 컨테이너를 삭제하고 새로 만들어도 볼륨(mydata)의 데이터가 유지됨을 확인

![alt text](image-4.png)
![alt text](<스크린샷 2026-11-05 164953.png>)

## 트러블슈팅

### 사례 1: 한글 깨짐
- 문제: 브라우저에서 한글 깨짐
- 원인: HTML 인코딩 미지정
- 해결: `<meta charset="UTF-8">` 추가

### 사례 2: 경로 변환 오류
- 문제: 바인드마운트 시 경로 오류
- 원인: Git Bash 자동 경로 변환
- 해결: `MSYS_NO_PATHCONV=1` 적용

### 사례 3: ! event not found
- 문제: echo에 느낌표 포함 시 오류
- 원인: Bash 특수문자(!) 해석
- 해결: 느낌표 제거

### 트러블슈팅 4. 8080 한글 깨짐
- 증상: 8082는 정상인데 8080만 한글 깨짐
- 원인: 8080은 charset 추가 전 빌드된 이미지 사용,
        8082는 바인드마운트로 수정본 반영
- 해결: docker build로 이미지 재빌드 후 컨테이너 재실행
- 교훈: 이미지는 빌드 시점 파일을 품으므로 수정 시 재빌드 필수

### 트러블슈팅 5: favicon.ico 404 로그

- 증상: docker logs 확인 시 /favicon.ico 요청에 404 발생
- 원인: 브라우저가 자동으로 사이트 아이콘(favicon)을 요청하나,
        이미지에 해당 파일이 없어 발생
- 해결: 정상 동작에 영향 없음을 확인. 필요 시 favicon.ico를
        추가하면 해소됨
- 교훈: 404가 항상 오류는 아니며, 로그를 통해 요청 흐름을 파악할 수 있음

### 트러블슈팅6: 포트 충돌 (port is already allocated)

**증상**
동일 포트(8080)로 두 번째 컨테이너 실행 시 아래 에러 발생:

docker: Error response from daemon: ... Bind for 0.0.0.0:8080 failed: port is already allocated

![alt text](<스크린샷 2026-11-14 130820-1.png>)

**원인**
이미 실행 중인 컨테이너(web1)가 8080 포트를 점유 중.
한 포트는 하나의 프로세스만 사용 가능하므로,
두 번째 컨테이너(web2)가 같은 포트를 바인딩하지 못해 발생.

**진단 방법**
netstat -ano | findstr :8080

→ LISTENING 상태의 PID 확인 (예: 7140 = Docker 점유)

![alt text](<스크린샷 2026-11-14 130921.png>)

**해결 방법**
1. 다른 포트로 실행
   docker run -d -p 8081:80 --name web2 nginx

2. 또는 기존 컨테이너 중지 후 재사용
   docker rm -f web1
   docker run -d -p 8080:80 --name web2 nginx

**배운 점**
- 포트는 한 번에 하나의 컨테이너만 바인딩 가능
- netstat -ano + PID로 점유 프로세스를 추적할 수 있음

## Dockerfile (커스텀 이미지)
```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
```
- `FROM nginx:alpine`: 가벼운 alpine 버전 nginx 공식 이미지를 베이스로 사용
- `COPY index.html ...`: 내 index.html을 nginx 웹 폴더에 복사
- 빌드 명령: `docker build -t mysite .`
- 트러블슈팅 4번처럼, HTML 수정 시 반드시 재빌드해야 반영됨

## Docker 환경 정보
- docker --version 결과: ![alt text](image.png)
- docker images 결과: ![alt text](image-1.png)
- docker ps -a 결과: ![alt text](image-2.png)
- docker stats 결과: ![alt text](image-3.png)

## 🔐 Linux 권한 실습
- chmod 644 testfile.txt (소유자 rw, 그룹/기타 r)
- chmod 755 testdir (소유자 rwx, 그룹/기타 rx)
- ls -l로 변경 전후 확인

## 터미널 기본 조작
![alt text](<스크린샷 2026-11-05 154607.png>)
![alt text](<스크린샷 2026-11-05 154631.png>)
![alt text](<스크린샷 2026-11-05 154654-1.png>)
![alt text](<스크린샷 2026-11-05 154709.png>)
![alt text](<스크린샷 2026-11-05 154726.png>)
![alt text](<스크린샷 2026-11-05 154757.png>)
![alt text](<스크린샷 2026-11-05 154822.png>)
![alt text](<스크린샷 2026-11-05 154837.png>)
![alt text](<스크린샷 2026-11-05 154859.png>)
![alt text](<스크린샷 2026-11-05 154920.png>)
![alt text](<스크린샷 2026-11-05 154936.png>)
![alt text](<스크린샷 2026-11-05 154951.png>)

### hello-world
hello-world 실행

![alt text](<스크린샷 2026-11-05 162505.png>)
![alt text](<스크린샷 2026-11-05 162527.png>)

**실행 로그 (텍스트):**

```bash
$ docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.
...
```

### ubuntu
(위 4장 스크린샷)
- docker run -it ubuntu bash 로 컨테이너 진입 ![alt text](<스크린샷 2026-11-05 160244.png>)
- ls 로 내부 리눅스 구조 확인 ![alt text](<스크린샷 2026-11-05 160316.png>)
- echo 로 명령 실행 후 exit ![alt text](<스크린샷 2026-11-05 160402.png>) ![alt text](<스크린샷 2026-11-05 160425.png>)

### attach vs exec 차이
- attach: 기존 메인 프로세스에 연결. exit 하면 컨테이너도 멈춤
- exec: 새 프로세스를 실행. exit 해도 컨테이너는 계속 실행됨

## 포트 접속 증거
![alt text](<스크린샷 2026-11-05 161111.png>)

### docker info 결과
![alt text](image-6.png)
![alt text](image-7.png)
![alt text](image-8.png)
![alt text](image-9.png)
![alt text](image-10.png)
![alt text](image-11.png)
![alt text](image-12.png)
![alt text](image-13.png)
![alt text](image-14.png)
![alt text](image-15.png)
![alt text](image-16.png)
![alt text](image-21.png) 
-> Docker 데몬 정상 작동 확인

`docker info | grep -A 10 "Server:"` 명령으로
데몬(Server) 정보를 확인했다.

- **Server** 섹션이 에러 없이 출력됨 → 데몬 정상 연결
- **Running: 1** → 컨테이너 1개 작동 중
- **Server Version: 29.9.2** → 엔진 정상 구동


### docker logs 결과
![alt text](image-17.png)

## #1 파일/폴더 생성·이동 기록

```
$ mkdir demo
$ touch demo/sample.txt
$ ls -l demo/
```
![alt text](image-18.png)

```
$ mv demo/sample.txt .
$ ls -l sample.txt
```
![alt text](image-19.png)


## #2 chmod 전후 비교

![alt text](image-20.png)

chmod 명령으로 파일 권한 변경을 실행함.
- chmod 755 : 소유자(rwx), 그룹(r-x), 기타(r-x)
- chmod 644 : 소유자(rw-), 그룹(r--), 기타(r--)

※ 참고: 본 실습은 Windows(Git Bash/UCRT64) 환경에서 진행되어
   NTFS 파일시스템 특성상 rwx 권한 변화가 화면에 반영되지 않음.
   리눅스 환경에서는 chmod 755 적용 시 -rwxr-xr-x 로 표시됨.

## Docker 빌드 결과

```
docker build -t mysite .

[+] Building 5.5s (7/7) FINISHED
 => [1/2] FROM nginx:alpine
 => [2/2] COPY index.html /usr/share/nginx/html/index.html
 => exporting to image
 => naming to docker.io/library/mysite:latest

→ 빌드 성공, mysite 이미지 생성 완료
```

## 생성된 이미지 확인
![alt text](image-23.png)

## Git 설정 확인

### git config --list

![alt text](스크린샷.png)


### git remote -v

![alt text](image-25.png)

## 📁 디렉토리 구성 및 폴더 역할

본 프로젝트는 **문서 · 빌드 설정 · 실습 산출물**을 분리하는 기준으로
디렉토리를 구성했습니다.

| 경로 | 역할 | 설계 기준 |
|------|------|----------|
| `Dockerfile` | nginx 이미지 빌드 정의 | 빌드 설정 파일은 프로젝트 루트에 배치 |
| `index.html` | nginx가 서빙하는 메인 웹 페이지 | 정적 리소스는 루트에 배치 |
| `README.md` | 프로젝트 문서 및 실습 증거 | 문서는 최상단에 두어 접근성 확보 |
| `demo/` | mkdir/mv 실습 시 생성·이동에 사용한 폴더 | #1 파일 생성·이동 실습 과정에서 생성 |
| `practice/` | 파일 생성·이동 명령 실습 공간 (b.txt, d.txt) | 실습 산출물은 별도 폴더로 격리 |
| `testdir/` | mkdir/mv 실습 결과물 보관 | 테스트 파일을 루트와 분리 |
| `image-*.png` | README에 첨부한 실행 증거 스크린샷 | 증거 이미지는 루트에서 참조 |

**설계 기준 요약**
- 빌드 설정(Dockerfile)과 문서(README)는 루트에 배치해 가시성 확보
- 실습·테스트 산출물은 `demo/`, `practice/`, `testdir/`로 격리해
  루트 디렉토리를 깔끔하게 유지

## 🌐 네트워크 네임스페이스 & 포트포워딩

### 네트워크 네임스페이스란?
컨테이너를 **격리**시키는 리눅스 커널 기술입니다.
각 컨테이너는 독립된 네트워크 공간(IP, 포트)을 가지므로,
서로 간섭 없이 실행됩니다.

- 호스트와 컨테이너의 네트워크가 분리됨
- 컨테이너끼리도 서로 격리됨

### 포트포워딩이 필요한 이유
네임스페이스로 컨테이너 네트워크가 **격리**되어 있어,
외부(브라우저)에서 컨테이너에 바로 접근할 수 없습니다.
따라서 **호스트 포트와 컨테이너 포트를 연결**해야 합니다.

```bash
docker run -p 8080:80 nginx
#              ↑    ↑
#           호스트  컨테이너
```

## 절대경로 / 상대경로 사용 기준

### 📌 개념 정리

| 구분 | 사용 기준 (언제 쓰나) | 예시 |
|------|----------------------|------|
| 절대경로 | 현재 위치와 상관없이 특정 위치를 한 번에 지정할 때 | `cd /c/test/교육장/site` |
| 상대경로 | 현재 위치를 기준으로 이동할 때 | `cd ..` , `cd site` |

### 📌 요약
- **절대경로**: `/` 로 시작하는 전체 경로. 어디에 있든 결과가 같음.
- **상대경로**: 현재 디렉터리(`.`) 기준으로 이동. 현재 위치에 따라 결과가 달라짐.

### 📌 실습 증거
![alt text](image-24.png)

- 절대경로 이동: `cd /c/test/교육장/site` → `pwd` 로 위치 확인
- 상대경로 이동: `cd ..` → `cd site` 로 이동

