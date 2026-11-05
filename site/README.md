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

## 권한 실습 (전/후 비교)
![alt text](<스크린샷 2026-11-05 155722.png>)
![alt text](<스크린샷 2026-11-05 155839.png>)

### hello-world
![alt text](<스크린샷 2026-11-05 162505.png>)
![alt text](<스크린샷 2026-11-05 162527.png>)

### ubuntu
(위 4장 스크린샷)
- docker run -it ubuntu bash 로 컨테이너 진입 ![alt text](<스크린샷 2026-11-05 160244.png>)
- ls 로 내부 리눅스 구조 확인 ![alt text](<스크린샷 2026-11-05 160316.png>)
- echo 로 명령 실행 후 exit ![alt text](<스크린샷 2026-11-05 160402.png>) ![alt text](<스크린샷 2026-11-05 160425.png>)

### attach vs exec 차이
- attach: 기존 메인 프로세스에 연결. exit 하면 컨테이너도 멈춤
- exec: 새 프로세스를 실행. exit 해도 컨테이너는 계속 실행됨
## Git 설정 확인
![alt text](<스크린샷 2026-11-05 160801.png>)

## 포트 접속 증거
![alt text](<스크린샷 2026-11-05 161111.png>)