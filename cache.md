# Concept

## Region

- Global
    - Redis = Single Thread. When Redis server is dead…
- Local
    - Need all to all replication or invalidate cache manually to sync each other
- Where to apply
    - DB > Session (SSO?) > App Data


# DB/JCR/WS Caching

## Operations

- Read
    - idempotent. cache! (But all SELECT operations OK?)
    - Cache-aside (Lazy loading) = Read-through
- Create, Update, Delete
    - Write-through

## Issues

- TTL (Time to live)
- WS → Session Scope
- Parameter 변화에 따른 Dynamic Caching : ValueMap hashing?
- Distributed / Shared Cache Topology? → 그럴 바엔 Global caching을 하지


# Java Caching Libraries

- JCache (JSR-107) : [https://www.jcp.org/en/jsr/detail?id=107](https://www.jcp.org/en/jsr/detail?id=107)
- Apache Commons JCS : [https://commons.apache.org/proper/commons-jcs/](https://commons.apache.org/proper/commons-jcs/)
- Google Guava Cache : [https://github.com/google/guava/wiki/CachesExplained](https://github.com/google/guava/wiki/CachesExplained)
- Ehcache : [http://www.ehcache.org](http://www.ehcache.org)
- JCache >= JCS, Guava > Ehcache
- SELECT Operations 중 조회량은 높고 변동량은 낮은 것부터 하나씩 적용
- 적용 전/후 성능 변화를 측정할 수 있는 방안 필요 (제니퍼로 스냅샷?)
- Annotation 사용 방법과 필요한 라이브러리
