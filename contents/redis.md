# Redis 정리
## Redis 캐시로 사용하기
`Cache`란?)

- 데이터의 원래 소스보다 더 빠르고 효율적으로 엑세스할 수 있는 임시 데이터 저장소

`Redis`는?)

- 단순한 key-value 구조로 저장하는 캐시
- In-memory 저장소 (평균 작업 속도 < 1 ms)

### 캐싱전략(읽기)

- 데이터베이스에 접근하기에 앞서 Redis에 먼저 접근하여 필요한 데이터를 빠르게 가져온다.(Cache Hit)
- Redis에 필요한 자료가 없을 경우(Cache Miss) 데이터베이스에 접근하여 데이터를 가져오도록 한다. (Lazy Loading)

### 특징

- Cache Miss가 다량 발생할 경우 해당 커넥션이 모두 데이터베이스로 붙으면서 부하가 발생할 수 있다.
- 데이터베이스에 신규 데이터나 기존 데이터를 미리 Redis에 올려 두는 Cache Warming 작업을 통해 부하 발생 가능성을 미리 방지

### 캐싱전략(쓰기)

- 데이터 쓰기는 데이터베이스에만 적용하는 `Write-Around` 방식과 Redis와 데이터베이스에 모두 저장하는 `Write-Through` 방식이 있다.

### 특징

- Write-Around 방식
    - Redis의 Cache Miss가 발생했을 경우에만 데이터베이스에서 데이터를 끌어오므로 Redis 데이터의 최신화 이슈가 발생할 수 있어 이를 위한 업데이트 전략이 필요함
- Write-Through 방식
    - Redis와 데이터베이스 간 데이터 일관성을 확보할 수 있다.
    - 쓰기 성능이 Write-Around에 비해 떨어진다.
    - 더이상 Hit가 발생하지 않는 데이터를 장시간 보유하여 리소스 낭비로 이어질 수 있으며, 이를 위한 Expire time 설정이 필요하다.

## Redis의 데이터 타입

### 종류

- Strings, Bitmaps, Lists, Hashes, Sets, Sorted Sets, HyperLogLogs, Streams

### 특징

- Strings: 기본 문자열
- Bitmaps: string의 변형, bit 단위 연산 가능
- Lists: 큐로 사용하기 적절함
- Hashes: 딕셔너리(오브젝트) 형태로 저장
- Sets: 집합(중복 불가)
- Sorted Sets: { member: score } 형태로 저장되는 집합으로 score순으로 정렬되어 저장된다. score가 같을 경우 member의 사전 순으로 정렬된다.
- HyperLogLogs: 중복되지 않는 값의 개수를 카운트할 때 사용
- Streams: log 저장에 주로 사용되는 append only 자료구조, kafka와 비슷하게 동작

### 활용

#### Counting
- `String`: INCR, INCRBY 등을 사용하여 값을 증가시키는 방식으로 카운팅을 한다.  

      redis > SET score:a 10
      ”OK”
      
      redis > INCR score:a
      (integer) 11
      
      redis > INCRBY score:a 4
      (integer) 15
    
- `Bitmap`: value로 오프셋 값을 주면 해당 비트의 위치에 0 또는 1값을 설정한다. 데이터를 정수로 변형할 수 있다면 오프셋으로 사용이 가능하며, 해당 자리의 값을 1로 바꿔 최종적으로 1인 값이 몇 개인지 카운팅하는 방식이다.  
        
      redis > SETBIT visitors:20240207 {id} 1
      (integer) 0
      
      redis > BITCOUNT visitors:20240207
      (integer) 1
        
  visitors:20240207라는 key에 오프셋(id)을 지정하여 값을 1로 주었다. 예시에서는 오프셋으로 id값을 준다고 가정했다.  
  
  당일에 천만 명이 방문했다 하더라도 천만 개의 비트가 차지하는 용량은 약 1.2MB에 해당하여 저장 공간을 절약할 수 있다.

- `HyperLogLogs`: set과 비슷하게 동작하지만 데이터 저장방식이 달라 차지하는 용량이 12KB로 고정된다. 그렇다보니 저장 공간 저장에 유리하여 대용량 데이터에 대한 카운팅에 활용된다. 하지만 저장된 데이터를 다시 확인할 수는 없으며 오로지 카운팅만 가능하다.  
        
      redis > PFADD crawled:20240207 “192.168.0.1”
      (integer) 1
      …
      
      redis > PFCOUNT crawled: 20240207
      (integer) 3546223
        
  위 예시에서는 해당 날짜에 크롤링한 IP의 개수가 몇 개인지 카운팅하는 예시이다.

#### Messaging
- `List`: Blocking 기능을 이용해 Event Queue로 사용이 가능하다. BRPOP과 LPUSH/RPUSH를 사용하지만 LPUSHX / RPUSHX를 사용하면 key가 존재할 때만 데이터를 push하게 할 수 있다.  

      client A
      redis > BRPOP myqueue 0

      client B
      redis > LPUSH myqueue "hi

      client A
      1) "myqueue"
      2) "hi"
      (26.54s)

  client A가 BRPOP을 통해 myqueue key에 대한 리스닝을 실행하면 해당 key에 push된 데이터를 즉시 가져오는 메시징이 가능하다.  

  또한 LPUSHX / RPUSHX를 사용하면 key가 존재할 때만 데이터를 push한다. 예를 들어 트위터나 인스타그램과 같이 유저별 Timeline이 존재하는 서비스에서 각 유저의 Timeline:{id}에 팔로우한 유저의 피드를 list에 미리 캐싱해 둘 수 있을 것이다. 이 때 서비스를 자주 사용하지 않는 유저는 Expire Time으로 인해 Timeline:{id}이 Redis의 key로 존재하지 않게 하여 관련 피드를 미리 push해두는 것을 방지할 수 있다. 이렇게 하면 저장 공간을 효율적으로 다룰 수 있다.

- `Stream`: '*'를 사용하여 데이터 저장 시간을 ID로 설정할 수 있어 로그를 저장하기 적절하다. append-only에 해당하고, 시간 범위로 검색, 신규 데이터 following, consumer group별 데이터 수신도 가능하다.  

      redis > XADD mystream * sensor-id 1234 temp 19.8
      "17252343562-0"

  '*'을 지정하면 mstime-sequenceNumber 형태로 저장된다.

## Redis 운영

### 데이터 영구 저장하기
- Redis를 캐시 이외의 용도로 사용한다면 적절한 데이터 백업이 필요할 것이다.
  
#### AOF 방식
- Append Only File의 약자로, 지금까지 입력된 커맨드를 그대로 저장하는 방식이다.
- 신규 key:value가 저장된 것부터 해당 key값이 변경, 제거되는 모든 과정을 담는다.
- RDB보다 대체적으로 용량이 커지므로 압축 또는 재작성의 과정이 추가적으로 필요하다.
- redis.conf의 auto-aof-rewrite-percentage옵션을 통해 파일 크기를 기준으로 자동 저장하거나 BGREWRITEAOF 커멘드로 수동 저장

#### RDB 방식
- Snapshot 방식으로 파일을 저장한다.
- AOF와 달리 최종 key:value만 기록하며 변천사는 기록하지 않는다.
- redis.conf의 SAVE옵션을 통해 시간 기준으로 자동 저장하거나 BGSAVE 커멘드로 수동 저장

### 선택 기준
- RDB: 어느정도의 데이터 유실이 허용될 경우 사용, SAVE 100 1로 설정하면 100초 간 1회 이상의 key변경이 발생할 경우에 저장한다. 잦은 저장은 성능에 영향을 줄 수 있으므로 적절한 값 설정이 필요하다.
- AOF: 장애 직전까지의 데이터 보장이 필요할 경우 사용, APPENDFSYNC값이 디폴트(everysec) 설정된 경우 최대 1초 사이의 데이터가 유실될 수 있음

### Tip
- key 조회 시 keys * 사용 금지 -> scan 0 사용할 것
  - keys의 경우 모든 key를 메모리에 올리려 하므로 프로세스가 길어져 블로킹이 발생할 수 있다.
  - scan의 경우 cursor라는 개념을 통해 부분적으로 key를 로딩하며 이 과정을 재귀 형태로 탐색하여 모든 키를 가져올 수 있다. 그러므로 프로세스를 분산할 수 있어 블로킹을 방지할 수 있다.

        redis > scan 0 MATCH key* COUNT 10
        1) "13"
        2)  1) "key1"
            2) "key14"
            3) "key4"
            4) "key7"
                .
                .
                .
 
        redis > scan 13 MATCH key* COUNT 10
            1) "0"
            2)  1) "key2"
                2) "key11"
                3) "key5"
                4) "key8"
    

    cursor 0을 시작으로 지정된 COUNT 개수만큼 조건에 맞게 탐색을 시작한다. 그러면 다음 순서에 해당하는 cursor인 "13"이 출력되었는데 이를 다음 scan의 cursor로 지정하면 이어서 탐색이 가능하다. 최종적으로 cursor값을 "0"을 반환했을 경우 전체 순회(full iteration)했다고 말한다. 이처럼 각 scan 커멘드가 각각의 프로세스가 되므로 그 틈 사이에 다른 커멘드가 실행될 여지가 생겨 블로킹이 방지되는 것이다.

    또한 보시다시피 scan 커멘드는 CLI가 아닌 스크립트에서 재귀함수 형태로 사용하는게 편리해 보인다.

- key 삭제 시 del 보다는 unlink 사용
  - del은 삭제 프로세스가 생성되므로 양이 많을수록 블로킹이 발생한다. unlink를 사용하면 백그라운드에서 삭제되므로 블로킹을 방지한다.
  - unlink는 해당 key를 삭제하여 key와 연결되어있던 메모리 공간이 붕~ 떠버리게 만드는 것이다. 더이상 해당 메모리 공간에 들어갈 루트가 없으므로 Garbage Collector가 이를 발견하면 메모리 해제 작업을 하는 비동기 방식으로 처리된다. 문제는 발견되기 전까지는 해당 메모리 공간이 유지되므로 Garbage Collector가 "언제 가져가느냐"가 중요한 관건이다. 논리적인 메모리 사용량과 물리적인 메모리 사용량이 일시적으로 격차가 발생하여 문제가 될 수 있기 때문이다. 그러므로 unlink를 통한 실제 메모리 공간 확보에는 비동기 처리 시간을 고려할 필요가 있다.
 
- STOP-WRITES-ON-BGSAVE-ERROR = NO
  - 해당 기능은 RDB 파일 저장에 실패하면 write기능을 차단함을 의미한다. 디폴트로 YES가 설정되어 있다.
  - 백업의 누락을 방지하기 위한 의도가 담겨있지만 오히려 write 차단으로 발생하는 서비스 장애가 더 치명적이므로 NO로 설정하고, RDB 저장 모니터링 환경을 갖추는 것이 좋다.

- MAXMEMORY-POLICY = ALLKEYS-LRU
  - redis의 메모리가 가득찼을 때 따를 정책을 선택한다.
  - 디폴트는 noeviction로 더이상 새로운 키를 저장하지 않는 정책이다. 이 또한 치명적인 서비스 장애로 이어질 수 있다.
  - allkeys-lru는 오랫동안 사용되지 않은 순으로 키를 삭제하는 정책이다. 메모리 용량에 따라 자주 사용되는 편에 속하던 key가 삭제되어 잦은 Cache Miss가 발생할 수도 있겠지만 이는 scale-up하면 해결될 문제이며 적어도 key가 생성되지 않는 불상사는 막을 수 있다.
  - 위 정책에 앞서 서비스 환경에 적합한 Expire Time을 설정하는 것이 중요하다.

- Cache Stampede
  - TTL 설정으로 key가 막 Expired 된 시점부터 redis에 key가 다시 생성되기 까지의 기간 사이에 여러 Application이 Expired된 key에 접근하려 한다면 그 여러 App이 모두 DB에 read를 시도하는 Duplicate Read가 발생하고, 이어서 새로 redis에 캐싱하려는 Duplicate Write까지 발생하게 된다. 위 과정은 불필요한 반복작업에 해당하며 성능 저하에 영향을 미치면서 장애로 이어질 수 있다. 주로 이벤트 오픈과 같은 합법적 DDos 공격을 받는 시점에 Expired 된 경우를 상상해보면 끔찍하다. 이를 방지하기 위해 Expired 되는 시점을 잘 고려해야 하며 차라리 DDos가 예상되는 시점 직전에 새로 key를 write하는게 안전하겠다.

- 메모리 관리
  - unlink에서 언급한 바와 같이 논리적 메모리 사용량과 물리적 메모리 사용량에 차이가 발생할 수 있다. 주로 Expire Time나 특정 이유로 인해 한꺼번에 많은 양의 key가 unlink된다면 Garbage Collector는 일이 바빠진다. 그러므로 물리적 메모리 공간 확보에 딜레이가 발생할 수 있다.(단편화) 그러므로 모니터링 시 used_memory_rss를 참조해야 한다. 또한 이러한 문제를 해결하기 위한 방법으로 공식문서에서는 active-defrag값을 일시적으로 yes로 설정할 것을 제안한다. 이는 메모리 최적화 프로세스를 백그라운드에서 실행함을 의미하는데 이또한 CPU자원을 차지하기 때문에 redis의 성능을 고려했을 때 단편화와 같은 특수한 상황에서 일시적으로만 적용할 것을 권장하는 것이다.

---

## Install Redis on macOS
[공식문서 참조](https://redis.io/docs/getting-started/installation/install-redis-on-mac-os/)

공식문서에서 cli 사용 예시도 찾아볼 수 있다.  
[cli 참조](https://redis.io/docs/ui/cli/)

```zsh
# 기본 cli 참고
brew services start redis
brew services info redis
brew services stop redis

# running Redis on cli
redis-cli
```

### Redis Cloud env 설정
[블로그 참조](https://inpa.tistory.com/entry/REDIS-%F0%9F%93%9A-Redis%EB%A5%BC-%ED%81%B4%EB%9D%BC%EC%9A%B0%EB%93%9C%EB%A1%9C-%EC%82%AC%EC%9A%A9%ED%95%98%EC%9E%90-Redislabs)

AWS의 elastic cache를 통해 Redis를 사용할 수 있음. 하지만 요금 발생  

Redis 공식 사이트에서 30mb 한정 무료 사용이 가능함  
[사이트 참조](https://redis.com/redis-enterprise-cloud/overview/)  

## 저장
메모리 기반이므로 기본적으로 서버가 종료되면 데이터는 날아간다. 이를 보완하기 위해 RDB과 AOF(Append-Only File) 기능이 있다. 후자는 logging에 해당하며 이를 기반으로 서버 복원이 가능하다.

## HOST 및 기타 설정 변경
vim /usr/local/etc/redis.conf


#### 참조
[참조 영상](https://www.youtube.com/watch?v=92NizoBL4uA)

