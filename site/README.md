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
$ MSYS_NO_PATHCONV=1 docker run -d --name vol-test4 -v mydata:/usr/share/nginx/html nginx
$ MSYS_NO_PATHCONV=1 docker exec vol-test4 cat /usr/share/nginx/html/test.txt
영속성 성공
```
→ 컨테이너 삭제 후에도 데이터 유지 확인

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