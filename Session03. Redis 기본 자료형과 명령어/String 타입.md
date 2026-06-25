# Redis String 명령어 모음

> 강의노트 Ch.03 — Redis 기본 자료형과 명령어 / String

---

## 1. 기본 명령어

```bash
# SET — 값 저장
127.0.0.1:6379> SET user:1:name "Kim"
# OK

# GET — 값 조회
127.0.0.1:6379> GET user:1:name
# "Kim"

# 존재하지 않는 키 조회
127.0.0.1:6379> GET user:999:name
# (nil)

# MSET — 여러 키-값 한번에 저장
127.0.0.1:6379> MSET user:1:name "Kim" user:2:name "Lee" user:3:name "Park"
# OK

# MGET — 여러 키 한번에 조회
127.0.0.1:6379> MGET user:1:name user:2:name user:3:name
# 1) "Kim"
# 2) "Lee"
# 3) "Park"

# APPEND — 값 뒤에 문자열 추가
127.0.0.1:6379> SET greeting "Hello"
127.0.0.1:6379> APPEND greeting " World"
# (integer) 11  ← 추가 후 전체 길이 반환
127.0.0.1:6379> GET greeting
# "Hello World"

# STRLEN — 값의 길이 확인 (바이트 수)
127.0.0.1:6379> STRLEN greeting
# (integer) 11

# SETNX — 키가 없을 때만 저장 (deprecated → SET NX 권장)
127.0.0.1:6379> SETNX user:1:name "Kim"
# (integer) 1  ← 저장 성공
127.0.0.1:6379> SETNX user:1:name "Lee"
# (integer) 0  ← 저장 실패 (키가 이미 존재)
```

> ⚠️ `STRLEN`은 문자 수가 아닌 **바이트 수**를 반환합니다. 한글 1글자 = UTF-8 기준 3바이트.

---

## 2. 숫자 연산

```bash
# INCR — 1 증가
127.0.0.1:6379> SET counter 100
127.0.0.1:6379> INCR counter
# (integer) 101

# DECR — 1 감소
127.0.0.1:6379> DECR counter
# (integer) 100

# INCRBY — 지정한 수만큼 증가
127.0.0.1:6379> INCRBY counter 10
# (integer) 110

# DECRBY — 지정한 수만큼 감소
127.0.0.1:6379> DECRBY counter 5
# (integer) 105
```

---

## 3. SET 명령어 옵션

```bash
# EX — 초 단위 만료 시간 설정
127.0.0.1:6379> SET session:abc "user:1" EX 60
# OK

# TTL — 남은 만료 시간 확인
127.0.0.1:6379> TTL session:abc
# (integer) 58

# NX — 키가 없을 때만 저장
127.0.0.1:6379> SET user:1:name "Kim" NX
# OK    ← 저장 성공
127.0.0.1:6379> SET user:1:name "Lee" NX
# (nil) ← 저장 실패 (키가 이미 존재)

# EX + NX — 만료 시간 + 조건부 저장 (분산락에 활용)
127.0.0.1:6379> SET lock:order:123 "server-1" EX 30 NX
# OK    ← 락 획득 성공
127.0.0.1:6379> SET lock:order:123 "server-2" EX 30 NX
# (nil) ← 락 획득 실패

# KEEPTTL — 값만 변경, 기존 만료 시간 유지
127.0.0.1:6379> SET session:abc "user:1" EX 60
127.0.0.1:6379> SET session:abc "user:2" KEEPTTL
127.0.0.1:6379> TTL session:abc
# (integer) 52  ← 기존 만료 시간 유지
```

> ⚠️ `KEEPTTL` 없이 SET을 다시 실행하면 만료 시간이 사라집니다.

---

## 4. 실무 활용 예시

```bash
# 캐시 — 상품 정보 10분 캐시
127.0.0.1:6379> SET product:100 "{name:'노트북', price:1500000}" EX 600
# OK

# 카운터 — 페이지뷰
127.0.0.1:6379> INCR pageview:main
# (integer) 1

# 세션 토큰 — 30분 유지
127.0.0.1:6379> SET session:abc123 "user:1" EX 1800
# OK

# 분산락 — 30초 락
127.0.0.1:6379> SET lock:payment:order123 "server-1" EX 30 NX
# OK
```
