# 실습 환경 설정 명령어 모음

> 강의노트 Ch.02 — Redis 개요와 환경 구축

---

## 1. WSL2 설치 (Windows 전용)

CMD 또는 PowerShell을 **관리자 권한**으로 실행 후 입력합니다.

```powershell
# WSL2 설치 (Linux 배포판 없이 엔진만 설치)
wsl --install --no-distribution
```

> 설치 완료 후 **재부팅** 필수

```powershell
# 재부팅 후 WSL2 설치 확인
wsl --status
```

출력 예시:
```
기본 버전: 2
```

---

## 2. Docker 설치 확인

Rancher Desktop 설치 후 터미널에서 확인합니다.

```bash
# Docker CLI 정상 동작 확인
docker version
```

`Client`와 `Server` 정보가 모두 출력되면 준비 완료입니다.

---

## 3. Redis 컨테이너 실행

```bash
# Redis 컨테이너 실행 (로컬 실습용)
docker run --name my-redis -p 6379:6379 -d redis
```

```bash
# Redis 컨테이너 실행 (외부 접근 차단 — 서버/클라우드 환경 권장)
docker run -d --name my-redis -p 127.0.0.1:6379:6379 redis
```

---

## 4. 컨테이너 상태 확인

```bash
# 실행 중인 컨테이너 목록 확인
docker ps
```

```bash
# Redis 버전 확인
docker exec -it my-redis redis-server --version
```

---

## 5. Redis CLI 접속

```bash
# Redis CLI 접속
docker exec -it my-redis redis-cli
```

프롬프트가 `127.0.0.1:6379>` 로 바뀌면 접속 성공입니다.

```bash
# 연결 확인
127.0.0.1:6379> PING
# PONG
```

---

## 6. 실습 환경 정리

```bash
# 컨테이너만 삭제 (이미지 유지)
docker rm -f my-redis
```

```bash
# 컨테이너 + 이미지 완전 삭제
docker rm -f my-redis
docker rmi redis
```

> ⚠️ 컨테이너를 먼저 삭제한 후 이미지를 삭제해야 합니다.
