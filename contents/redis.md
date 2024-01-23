# Redis

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

## Redis Cloud env 설정
[블로그 참조](https://inpa.tistory.com/entry/REDIS-%F0%9F%93%9A-Redis%EB%A5%BC-%ED%81%B4%EB%9D%BC%EC%9A%B0%EB%93%9C%EB%A1%9C-%EC%82%AC%EC%9A%A9%ED%95%98%EC%9E%90-Redislabs)

AWS의 elastic cache를 통해 Redis를 사용할 수 있음. 하지만 요금 발생  

Redis 공식 사이트에서 30mb 한정 무료 사용이 가능함  
[사이트 참조](https://redis.com/redis-enterprise-cloud/overview/)  

## 저장
메모리 기반이므로 기본적으로 서버가 종료되면 데이터는 날아간다. 이를 보완하기 위해 스냅샷 기능과 AOF(Append-Only File) 기능이 있다. 후자는 logging에 해당하며 이를 기반으로 서버 복원이 가능하다고 한다.  

## HOST 및 기타 설정 변경
vim /usr/local/etc/redis.conf


## Node.js 환경에서 redis 사용

```zsh
# v3는 콜백 기반, v4는 프로미스 기반
npm i redis 
```





