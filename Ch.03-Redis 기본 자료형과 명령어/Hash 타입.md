# Redis Hash 명령어 모음

> 강의노트 Ch.03 — Redis 기본 자료형과 명령어 / Hash

---

## 1. 기본 명령어

```bash
# HSET — 필드 저장 (여러 개 동시 가능)
127.0.0.1:6379> HSET user:1 name "Kim" age "30" email "kim@example.com" city "Seoul"
# (integer) 4   ← 새로 추가된 필드 수

# 이미 존재하는 필드 수정
127.0.0.1:6379> HSET user:1 name "Lee"
# (integer) 0   ← 수정된 필드 (새로 추가된 필드 없음)

# HGET — 특정 필드 조회
127.0.0.1:6379> HGET user:1 name
# "Lee"

# 존재하지 않는 필드 조회
127.0.0.1:6379> HGET user:1 phone
# (nil)

# HMGET — 여러 필드 한번에 조회
127.0.0.1:6379> HMGET user:1 name age email
# 1) "Lee"
# 2) "30"
# 3) "kim@example.com"

# HGETALL — 모든 필드·값 조회
127.0.0.1:6379> HGETALL user:1
# 1) "name"
# 2) "Lee"
# 3) "age"
# 4) "30"
# 5) "email"
# 6) "kim@example.com"
# 7) "city"
# 8) "Seoul"

# HDEL — 필드 삭제
127.0.0.1:6379> HDEL user:1 email
# (integer) 1   ← 삭제된 필드 수

# 여러 필드 한번에 삭제
127.0.0.1:6379> HDEL user:1 age city
# (integer) 2

# HEXISTS — 필드 존재 여부 확인
127.0.0.1:6379> HEXISTS user:1 name
# (integer) 1   ← 존재함

127.0.0.1:6379> HEXISTS user:1 phone
# (integer) 0   ← 존재하지 않음
```

> ⚠️ 필드가 많은 Hash에서 `HGETALL`은 부하가 커집니다. 필요한 필드만 `HMGET`으로 조회하는 것을 권장합니다.

---

## 2. 필드 관리 명령어

```bash
# 실습 데이터 준비
127.0.0.1:6379> HSET user:1 name "Lee" age "30" email "lee@example.com"
# (integer) 3

# HKEYS — 모든 필드 이름 조회
127.0.0.1:6379> HKEYS user:1
# 1) "name"
# 2) "age"
# 3) "email"

# HVALS — 모든 값 조회
127.0.0.1:6379> HVALS user:1
# 1) "Lee"
# 2) "30"
# 3) "lee@example.com"

# HLEN — 필드 개수 확인
127.0.0.1:6379> HLEN user:1
# (integer) 3
```

---

## 3. 숫자 연산

```bash
# HINCRBY — 정수 증감
127.0.0.1:6379> HSET product:1 stock 100
127.0.0.1:6379> HINCRBY product:1 stock -1
# (integer) 99   ← 재고 1개 감소

127.0.0.1:6379> HINCRBY product:1 stock 10
# (integer) 109  ← 재고 10개 증가

# HINCRBYFLOAT — 부동소수점 증감
127.0.0.1:6379> HSET product:1 price 10.50
127.0.0.1:6379> HINCRBYFLOAT product:1 price 1.50
# "12"
```

> ⚠️ 금액처럼 정확도가 중요한 경우 정수로 변환(원 단위)하여 `HINCRBY`를 사용하는 것을 권장합니다.

---

## 4. 필드 단위 TTL (Redis 7.4+)

```bash
# 실습 데이터 준비
127.0.0.1:6379> HSET session:user1 token "abc123" device "mobile"
# (integer) 2

# HEXPIRE — 필드에 만료 시간 설정 (초 단위)
127.0.0.1:6379> HEXPIRE session:user1 60 FIELDS 1 token
# 1) (integer) 1   ← 만료 시간 설정 성공

# HTTL — 필드의 남은 만료 시간 확인
127.0.0.1:6379> HTTL session:user1 FIELDS 1 token
# 1) (integer) 58

# 만료 시간 없는 필드 확인
127.0.0.1:6379> HTTL session:user1 FIELDS 1 device
# 1) (integer) -1  ← 만료 시간 없음
```

> ⚠️ 필드 단위 TTL은 **Redis 7.4 이상**에서만 지원됩니다.

---

## 5. HSCAN — 대량 필드 조회

```bash
# 실습 데이터 준비
127.0.0.1:6379> HSET user:1 name "Lee" age "30" email "lee@example.com" city "Seoul" grade "VIP"

# 커서 0부터 시작
127.0.0.1:6379> HSCAN user:1 0
# 1) "0"           ← 다음 커서 (0이면 순회 완료)
# 2) 1) "name"
#    2) "Lee"
#    3) "age"
#    4) "30"
#    5) "email"
#    6) "lee@example.com"
#    7) "city"
#    8) "Seoul"
#    9) "grade"
#    10) "VIP"

# 패턴으로 필터링
127.0.0.1:6379> HSCAN user:1 0 MATCH *a*
# 1) "0"
# 2) 1) "name"
#    2) "Lee"
#    3) "grade"
#    4) "VIP"
```

> ⚠️ `COUNT`는 정확한 개수가 아니라 힌트입니다. 실제 반환되는 필드 수는 다를 수 있습니다.

---

## 6. 실무 활용 예시

```bash
# 사용자 프로필
127.0.0.1:6379> HSET user:1001 name "Kim" age "30" email "kim@example.com" grade "VIP"

# 등급만 업데이트
127.0.0.1:6379> HSET user:1001 grade "GOLD"

# 프로필 전체 조회
127.0.0.1:6379> HGETALL user:1001

# 상품 정보
127.0.0.1:6379> HSET product:500 name "노트북" price "1500000" stock "50" category "전자제품"

# 재고 감소
127.0.0.1:6379> HINCRBY product:500 stock -1

# 가격·재고만 조회
127.0.0.1:6379> HMGET product:500 price stock

# 실시간 통계 카운터
127.0.0.1:6379> HINCRBY stats:2025-01-01 pageview 1
127.0.0.1:6379> HINCRBY stats:2025-01-01 login 1
127.0.0.1:6379> HINCRBY stats:2025-01-01 purchase 1

# 하루 통계 전체 조회
127.0.0.1:6379> HGETALL stats:2025-01-01
```
