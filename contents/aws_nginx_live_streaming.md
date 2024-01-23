# AWS의 EC2, Nginx, OBS로 라이브 스트리밍 구현하기

라이브 스트리밍 프로젝트를 준비하는 과정에서 nginx로 스트리밍이 가능하다는 것을 알게 되었고, 크롬의 hls 실행 익스텐션으로 라이브 스트리밍을 시청하는데 성공했다.  
하루를 꼬박 보내버리면서 알게된 방법을 차근차근 정리해보았다.  

## AWS EC2에서 구현하기
프리티어인 t2.micro 인스턴스(ubuntu22.04)를 생성한다. 이후 sg(보안 그룹)의 인바운드 규칙에서 앞으로 사용할 여러 포트들을 열어줘야 한다.  
- 80포트(http)
- 8088포트(m3u8파일 읽기를 위한 포트)
- 1935포트(rtmp 프로토콜)

## 패키지 설치
EC2 우분투 서버에 접속 후 스트리밍을 위한 패키지 설치를 진행했다.  

```zsh
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get install nginx
sudo apt install libnginx-mod-rtmp # nginx와 rtmp를 위한 패키지
sudo apt install net-tools # ufw status 확인 시 필요
```

## 방화벽 설정  
ubuntu에서 기본적으로 제공하는 ufw 방화벽 설정은 사실 지금 하려는 프로젝트에 필수 요소는 아닌 듯 보였다.  
하지만 정확히 열어주고 싶은 특정 IP나 PORT만 열 수 있다는 점에서 네트워크 엑세스를 제한하여 보안 규칙을 세울 수 있다.

방화벽을 사용하려면 ufw를 사용하는데 기본적으로 disable(비활성화) 상태이므로 enable(활성화)해줄 필요가 있다.  
하지만 그 전에 앞서 우리가 사용할 포트를 allow 해준 뒤 활성화 하도록 하자

```zsh
sudo ufw allow 22/tcp # ssh 프로토콜
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp # https 프로토콜을 쓴다면
sudo ufw allow 1935/tcp
sudo ufw allow 8088/tcp
```

그 외에도 mySQL이나 Redis 등을 사용한다면 추가적으로 열어줘야 할 것이다. 이는 EC2 sg에도 마찬가지다.  

> 🚨 주의  
> 22/tcp 포트를 allow하는 것을 결코 잊어서는 안된다. 이를 설정하지 않은 채 ufw enable을 하게 되면 다시 ec2 접속이 불가하다.  
> 만일 이런 경우가 발생했을 경우 기존 EC2 인스턴스에서 EBS(하드)를 분리하고, 새 EC2 인스턴스 생성 후 기존 EBS를 붙인 뒤 접근하는 방식으로 문제를 해결해야 한다.  
> 그래서 기존 EBS에 들어가 ufw를 disable하는, 쉽게 말해 우회 접근하여 방화벽을 끄는 시도를 해야한다.  
> 방법은 여러 블로그에 소개되어 있지만 상당히 번거롭다는 것을 기억하자! 특히 프리티어에서 제공하는 EBS 용량이 30GB이므로 이를 넘어서는 순간 비용 발생한다는 점도 참고하자.  

추가적으로 node.js 프로젝트를 사용한다면 아마도 3000port를 사용할 것으로 예상된다.  
하지만 nginx를 적용한다면 80번 포트 또는 443 포트로 접근 시 3000포트로 리다이렉트 하는 방식으로의 설정이 필요할 것이다.  
npm start를 하면 3000포트를 nodejs가 점유하게 되므로 굳이 3000포트를 열어 둘 필요가 없어진다.  

```zsh
node.js 를 사용한다면 3000번 포트 막기
sudo ufw deny 3000/tcp
```

방화벽 설정을 모두 마쳤다면 `sudo ufw enable`을 입력하여 방화벽을 활성화 한다.  
이 때 22번 포트가 막혀있으면 EC2 접속이 끊어질 수도 있는데 괜찮겠냐? 는 질문이 나오지만 우리는 이미 22 번포트를 열어줬으므로 y를 입력해 준다.  
최종적으로 `sudo ufw status`를 통해 방화벽 설정을 확인하자

## nginx 설정
우리가 방화벽에서 열어준 포트로 접근하면 어떻게 할 것인지에 대한 설정이라고 이해하자  
설정을 위해서 우리가 수정해야 할 파일은 아래 두가지이다.

- sudo vi /etc/nginx/nginx.conf
- sudo vi /etc/nginx/sites-avaliable/default

위 두 파일(nginx.conf, defalut)은 sudo 권한이 있어야 편집이 가능하다.

먼저 1935 포트, rtmp 프로토콜 접근 시 해줄 일부터 설정해보자


```zsh
# /etc/nginx/nginx.conf

user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
        worker_connections 768;
        # multi_accept on;
}

# nginx rtmp 설정
rtmp {
        server {
                # rtmp 포트 번호
                listen 1935;

                # rtmp 가 4kb 블록으로 데이터 전송
                chunk_size 4096;

                # hls 형식으로 변환
                # 스트림 데이터가 기본적으로 디스크에 저장되지 않는다
                application live {
                        live on;
                        record off;

                        # hls
                        hls on;
                        hls_path /var/www/html/stream/hls;
                        hls_fragment 3s;
                        hls_playlist_length 5s;
                        #hls_cleanup off;
                 }
        }
}
```

`nginx rtmp 설정` 부분에 주목하자 기존 파일에 rtmp 부분을 추가하면 된다.  
내용을 간단하게 정리하자면 아래와 같다.  

- 1935포트에 대한 반응입니다.
- rtmp는 http와 같은 프로토콜 이름이며, 실시간 스트리밍 시 주로 사용합니다.
- application live에 접근하면 hls를 수행합니다.

여기서 hls가 무슨 기능인지 간단하게 정리하자면 아래와 같다.

- 실시간 스트리밍을 시작한 이후 일정 시간이 지나면 방영한 영상을 .ts파일로 조각내어 저장하기 시작합니다.
- 그리고 스트리밍 시청자는 이 .ts파일을 순차적으로 시청하게 됩니다. 엄연히 말해서 실시간 방송이 아닌(?) 느낌이 됩니다.
- 그리고 어느 시점에 어떤 .ts파일을 보여줘야 할지 가이딩 해주는 .m3u8 파일도 함께 저장한다.
- 이렇게 하면 실제 스트리밍과의 딜레이가 발생하게 되지만 서버는 저장된 파일을 웹 플레이어에서 재생되도록 하면 되므로 상대적으로 부담이 줄어든다.
- 여기서 상대적이라 함은 zoom과 같은 화상 회의에 적용하는 Web RTC의 peer to peer 방식과 비교했을 때를 의미한다.

위 코드에서 주목할 점은 먼저 `application live` 라는 부분이다. 여기서 'live' 라는 단어는 자유롭게 바꿀 수 있으며 만약 live로 지정했다면,    
나중에 OBS에 연결할 때 rtmp://EC2 서버 IP/live 와 같은 형태로 지정해야 한다. 즉 rtmp프로토콜로 해당 서버의 `/live` 가 application의 트리거가 된다.  

두번째로는 `hls_path /var/www/html/stream/hls;` 이다. hls의 조각파일들은 해당 경로에 저장하겠다는 의미이다. 이 경로는 ubuntu의 절대경로이다.

세번째로 hls의 세부 옵션 중 hls_fragment와 hls_playlist_length값을 주목할 필요가 있다.  
fragment는 chunk라고 불리는 각각의 .ts파일이 저장되는 영상의 길이를 설정한다. 위 예시처럼 3s로 설정된다면 3초짜리 영상이(3초 단위로) .ts(chunk)로 저장됨을 의미한다.  

그리고 playlist_length는 .ts파일을 삭제하지 않고 유지하는 시간에 관여한다. 참고로 hls은 .ts파일을 정기적으로 삭제하는 것이 default로 설정되어 있다. 스트리밍이 길어질수록 무한정 .ts파일이 쌓이는 것을 방지하기 위함이다. 이러한 로직에서 playlist_length는 삭제 주기에 관여한다.  

fragment를 30s로 설정하고 playlist_length를 60s로 설정한 결과 8번째 .ts파일 저장이 시작되었을 때 초반 3개의 .ts파일이 삭제되었다.  
대략적으로 계산해보면 30초 * 7개 = 210초 니까 3분 30초 분량이 ts파일이 쌓인 뒤 초반 1분 30초 가량의 ts를 삭제하여 2분 가량의 ts만 남겼다.  
정확한 삭제 로직은 알 수 없지만 결론적으로는 playlist_length를 1분으로 설정했더니 3분 30초 쯤 되는 시점에서 초반 1분 30초 분량을 삭제했다.  
결국 playlist_length로 설정한 시간보다 약 3배 가량 넉넉하게 시간이 지났을 즈음 초반 ts파일을 날린다고 볼 수 있겠다.  

마지막으로 hls_cleanup 옵션은 ts 자동 삭제를 정지하는 옵션이다. default는 on일 것으로 예상되며 off로 설정하면 ts를 결코 지우지 않는다.


```zsh
# sudo vi /etc/nginx/sites-available/default

server {
    listen 8088;

    location / {
		add_header Cache-Control no-cache;
		add_header 'Access-Control-Allow-Origin' '*';
		add_header 'Access-Control-Expose-Headers' 'Content-Length';

        # :8088 의 root 경로를 지정
        root /var/www/html/stream;

        types {
            application/vnd.apple.mpegurl m3u8;
        }
    }
}
```
default 파일에 위 코드를 추가해준다.  
이 코드에서 주목할 점은 root와 types이다. root는 서버IP(도메인):8088로 접속할 시 /var/www/html/stream 경로를 바라보도록 한다.  
그래서 이후 stream 폴더의 하위에 위치한 hls폴더 내 .m3u8파일에 접근하여 플레이 하도록 하는 용도로 활용될 것이다.  

또한 types는 .m3u8 파일을 application/vnd.apple.mpegurl라는 mimetype으로 제공되도록 하는 기능으로 라이브 스트리밍에 필요한 지정입니다.

이제 stream 폴더를 생성해주고 nginx를 실행 합니다.

```zsh
sudo service nginx start # nginx 실행
sudo systemctl reload nginx # nginx 재실행
```

> 🚨 주의  
> nginx 수정 시 세미콜론(;) 하나 제대로 안닫거나, 문자가 틀린다면 nginx는 정상적으로 실행되지 않습니다.  
> `sudo service nginx configtest`를 입력하여 nginx.conf 설정에 문제가 없는지 확인합니다.  
> `systemctl status nginx.service` 를 통해 현재 nginx가 실행 중인지, 문제가 있는지 등을 파악할 수 있습니다.

우선 라이브 스트리밍을 위한 백엔드에서의 설정은 끝이났다.  

## OBS 설정
OBS의 설정(Preference)에 들어가 방송탭(Stream)에 들어가면 서비스, 서버, 스트림키 이렇게 세 가지 입력란이 있을 것이다.  
- 서비스는 Custom 즉 '사용자 지정...'으로 설정
- 서버는 rtmp://issuebombom.store/live
- 스트림키는 자유 설정이 가능하나 서버에서 지정해 줄 필요가 있다.

위 설정에서 만약 스트림키를 'test'라고 설정했다면 hls 폴더 내 test.m3u8 파일이 생성된다.  
그리고 test.m3u8은 웹 스트림 플레이어에서 참조해야할 정보이다. 그러므로 서버에서 지정해주면 스트리머가 그대로 기입할 필요가 있다.

이렇게 설정한 뒤 OBS의 방송 시작을 클릭하면 EC2와 연결되면서 hls 관련 파일들을 저장하기 시작할 것이다.  
참고로 npm run dev나 npm start에 익숙하신 분들은 '서버 실행을 해야 되는 거 아닌가?' 라는 생각이 들 수 있을지도 모르겠다.  
하지만 이건 nginx와 OBS의 소통이므로 nodejs와 상관이 없으며, 우선 우리는 nodejs를 설치하지도 않았다.

## 웹 hls 플레이어로 스트리밍 방송 보기
OBS 스트리밍이 정상적으로 진행 중이라면 hls 전용 플레이어로 실시간 방송을 볼 수 있다.  
`http://서버IP 혹은 도메인주소:8088/hls/스트림키.m3u8` 주소를 hls 플레이어를 제공하는 사이트 혹은 Chrome Extention으로 적용해보면
실시간 방송을 그대로 볼 수 있을 것이다. 특히 도메인 주소를 등록했을 경우 '가비아'와 같은 곳에서의 DNS 설정 시 www 설정을 했는지 안했는지 잘 체크하기 바란다.

## 추가로 필요한 작업
아무래도 스트림키를 클라이언트에게 직접 제시하는 방법을 모색해야할 것 같다.  
또한 방송 시작 시 node 서버가 눈치채고 hls 플레이어를 실행할 수 있도록 하는 로직이 필요할 것 같다. 

참조 링크  
https://github.com/arut/nginx-rtmp-module/wiki  
https://github.com/arut/nginx-rtmp-module/wiki/Directives#notify  
https://realizetoday.tistory.com/entry/Live-Streaming-Server-%EA%B5%AC%EC%B6%95-5-Nginx-rtmp-module-%EC%84%A4%EC%B9%98-rtmp-to-hls  
https://www.digitalocean.com/community/tutorials/how-to-set-up-a-video-streaming-server-using-nginx-rtmp-on-ubuntu-20-04  
https://www.hostwinds.kr/tutorials/live-streaming-from-a-vps-with-nginx-rtmp  
https://qteveryday.tistory.com/371  
https://qteveryday.tistory.com/372  
https://3jini.tistory.com/245  
https://3jini.tistory.com/246  
https://webdir.tistory.com/206  
  