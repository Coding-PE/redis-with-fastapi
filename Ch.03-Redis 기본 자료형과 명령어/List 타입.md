# Redis List 명령어 모음

> 강의노트 Ch.03 — Redis 기본 자료형과 명령어 / List

---

## 1. 기본 명령어

```bash
# LPUSH — 왼쪽(Head)에 값 삽입
127.0.0.1:6379> LPUSH mylist "a"
# (integer) 1
127.0.0.1:6379> LPUSH mylist "b"
# (integer) 2
127.0.0.1:6379> LPUSH mylist "c"
# (integer) 3
# 현재 상태: [c, b, a]

# RPUSH — 오른쪽(Tail)에 값 삽입
127.0.0.1:6379> RPUSH mylist "d"
# (integer) 4
127.0.0.1:6379> RPUSH mylist "e"
# (integer) 5
# 현재 상태: [c, b, a, d, e]

# 여러 값을 한 번에 삽입
127.0.0.1:6379> RPUSH mylist "x" "y" "z"
# (integer) 8

# LRANGE — 범위 조회
127.0.0.1:6379> LRANGE mylist 0 -1      # 전체 조회
# 1) "c"
# 2) "b"
# 3) "a"
# 4) "d"
# 5) "e"

127.0.0.1:6379> LRANGE mylist 0 2       # 0~2번째 조회
# 1) "c"
# 2) "b"
# 3) "a"

127.0.0.1:6379> LRANGE mylist -3 -1     # 뒤에서 3개 조회
# 1) "a"
# 2) "d"
# 3) "e"

127.0.0.1:6379> LRANGE mylist -1 -1     # 마지막 요소만 조회
# 1) "e"

# LLEN — List 길이 확인
127.0.0.1:6379> LLEN mylist
# (integer) 5

# LPOP — 왼쪽에서 값 꺼내기
127.0.0.1:6379> LPOP mylist
# "c"

# RPOP — 오른쪽에서 값 꺼내기
127.0.0.1:6379> RPOP mylist
# "e"
# 현재 상태: [b, a, d]

# 여러 개를 한 번에 꺼내기
127.0.0.1:6379> LPOP mylist 2
# 1) "b"
# 2) "a"
```

> ⚠️ `LPOP`/`RPOP`은 꺼낸 값이 List에서 **삭제**됩니다. 단순 조회는 `LRANGE`를 사용하세요.

---

## 2. 실무 활용 패턴

```bash
# 큐 (Queue) — 작업 대기열 (FIFO)
# 작업 추가 (Producer)
127.0.0.1:6379> LPUSH job:queue "job:1"
# (integer) 1
127.0.0.1:6379> LPUSH job:queue "job:2"
# (integer) 2
127.0.0.1:6379> LPUSH job:queue "job:3"
# (integer) 3

# 작업 처리 (Consumer)
127.0.0.1:6379> RPOP job:queue
# "job:1"   ← 가장 먼저 들어온 작업이 먼저 나옴
127.0.0.1:6379> RPOP job:queue
# "job:2"

# 최근 본 상품 목록 — 최근 N개만 유지
127.0.0.1:6379> LPUSH recent:user:1 "product:101"
127.0.0.1:6379> LPUSH recent:user:1 "product:205"
127.0.0.1:6379> LPUSH recent:user:1 "product:340"
127.0.0.1:6379> LPUSH recent:user:1 "product:87"
127.0.0.1:6379> LPUSH recent:user:1 "product:512"

# 최근 5개만 유지
127.0.0.1:6379> LTRIM recent:user:1 0 4

127.0.0.1:6379> LRANGE recent:user:1 0 -1
# 1) "product:512"
# 2) "product:87"
# 3) "product:340"
# 4) "product:205"
# 5) "product:101"

# 새 상품 조회 시 LPUSH + LTRIM 함께 실행
127.0.0.1:6379> LPUSH recent:user:1 "product:999"
127.0.0.1:6379> LTRIM recent:user:1 0 4    # 가장 오래된 항목 자동 제거
```

---

## 3. 추가 명령어

```bash
# BLPOP / BRPOP — 블로킹 방식으로 값 꺼내기
# List가 비어 있어도 30초 대기
127.0.0.1:6379> BRPOP job:queue 30

# LINSERT — 특정 값의 앞/뒤에 삽입
127.0.0.1:6379> RPUSH mylist "a" "b" "d"
127.0.0.1:6379> LINSERT mylist BEFORE "d" "c"
# (integer) 4
127.0.0.1:6379> LRANGE mylist 0 -1
# 1) "a"
# 2) "b"
# 3) "c"
# 4) "d"

# LSET — 특정 인덱스 값 변경
127.0.0.1:6379> LSET mylist 0 "z"
# OK
127.0.0.1:6379> LRANGE mylist 0 -1
# 1) "z"
# 2) "b"
# 3) "c"
# 4) "d"

# LREM — 특정 값 삭제
127.0.0.1:6379> RPUSH mylist "a" "b" "a" "c" "a"

# 양수: head부터 2개 삭제
127.0.0.1:6379> LREM mylist 2 "a"
# (integer) 2
127.0.0.1:6379> LRANGE mylist 0 -1
# 1) "b"
# 2) "c"
# 3) "a"   ← 마지막 "a"는 남아 있음

# 음수: tail부터 2개 삭제
127.0.0.1:6379> RPUSH mylist2 "a" "b" "a" "c" "a"
127.0.0.1:6379> LREM mylist2 -2 "a"
# (integer) 2
127.0.0.1:6379> LRANGE mylist2 0 -1
# 1) "a"   ← 첫 번째 "a"는 남아 있음
# 2) "b"
# 3) "c"

# 0: 전체 삭제
127.0.0.1:6379> RPUSH mylist3 "a" "b" "a" "c" "a"
127.0.0.1:6379> LREM mylist3 0 "a"
# (integer) 3
127.0.0.1:6379> LRANGE mylist3 0 -1
# 1) "b"
# 2) "c"

# LPOS — 특정 값의 인덱스 검색
127.0.0.1:6379> RPUSH mylist "a" "b" "c" "b"
127.0.0.1:6379> LPOS mylist "b"
# (integer) 1   ← 첫 번째 "b"의 인덱스
```
