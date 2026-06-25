# Redis CLI 기본 조작 명령어 모음

> 강의노트 Ch.02 — Redis CLI 기본 조작

---

## 1. Redis CLI 접속 / 종료

```bash
# Redis CLI 접속
docker exec -it my-redis redis-cli
```

```bash
# Redis CLI 종료
127.0.0.1:6379> exit
# 또는
127.0.0.1:6379> quit
```

> ⚠️ CLI를 종료해도 Redis 서버(컨테이너)는 계속 실행 중입니다.

---

## 2. 연결 확인 (PING)

```bash
127.0.0.1:6379> PING
# PONG

# 메시지 지정
127.0.0.1:6379> PING "안녕하세요"
# "안녕하세요"
```

---

## 3. 데이터 기본 조작

```bash
# SET — 데이터 저장
127.0.0.1:6379> SET user:1:name "Kim"
# OK

# GET — 데이터 조회
127.0.0.1:6379> GET user:1:name
# "Kim"

# 존재하지 않는 키 조회
127.0.0.1:6379> GET user:999:name
# (nil)

# DEL — 데이터 삭제
127.0.0.1:6379> DEL user:1:name
# (integer) 1

# 여러 키 한 번에 삭제
127.0.0.1:6379> DEL user:1:name user:2:name user:3:name
# (integer) 3

# EXISTS — 키 존재 여부 확인
127.0.0.1:6379> EXISTS user:1:name
# (integer) 1  → 존재함

127.0.0.1:6379> EXISTS user:999:name
# (integer) 0  → 존재하지 않음
```

---

## 4. 키 관리

```bash
# KEYS — 전체 키 조회 (⚠️ 실습 환경에서만 사용)
127.0.0.1:6379> KEYS *

# 패턴으로 검색
127.0.0.1:6379> KEYS user:*

# DBSIZE — 키 개수 확인
127.0.0.1:6379> DBSIZE
```

> ⚠️ `KEYS *`는 운영 환경에서 사용 금지. 키가 수백만 개인 경우 서버 전체가 멈출 수 있습니다.

---

## 5. 서버 관리

```bash
# INFO — 서버 전체 상태 확인
127.0.0.1:6379> INFO

# 섹션 지정
127.0.0.1:6379> INFO memory
127.0.0.1:6379> INFO server

# FLUSHDB — 현재 DB 데이터 전체 삭제 (⚠️ 실습 환경에서만 사용)
127.0.0.1:6379> FLUSHDB

# FLUSHALL — 전체 DB 데이터 삭제 (⚠️ 실습 환경에서만 사용)
127.0.0.1:6379> FLUSHALL
```

> ⚠️ `FLUSHDB` / `FLUSHALL` 은 삭제 후 복구 불가합니다.
