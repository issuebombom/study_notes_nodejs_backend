# Nest.js에서 외부 API에 접근하기

해당 과정은 MSA 학습 과정에서 메시징, 큐 기반 통신을 구현하기 전, http 통신으로 서버간 소통하는 과정을 우선 구현해보고자 진행된 결과이다.

## Python 서버와 Node.js 서버 간 통신하기

아래 과정을 간단하게 구현한다.

1. pytube라는 파이썬 패키지를 사용하면 유튜브 링크 입력 시 해당 영상 정보를 받을 수 있다.
2. 1번 기능을 하는 서버를 외부 API로 취급하여 Node.js 서버에서 url을 보내면 관련 정보를 전달 받도록 한다.

### 파이썬 pytube 서버

```python
# app.py
from flask import Flask, request
from werkzeug.exceptions import BadRequest
from utils import YoutubeAudioExtractor

app = Flask(__name__)

@app.route('/urls', methods=['POST'])
def get_url_information():
    req = request.get_json()

    try:
        urls = req['urls'] # string
        youtube = YoutubeAudioExtractor(urls)
        urls_information = youtube.extract_url_information()

        return urls_information # jsonify없이 잘 전달됨

    except Exception as e:
        return { 'error': f'{e}' }

if __name__ == '__main__':
    app.run(host='127.0.0.1', port=5000, debug=True)

```

파이썬 Flask를 활용하면 매우 간단하게 end-point를 생성할 수 있다.  
로컬서버의 5000포트로 `/urls` 경로를 통해 유튜브 링크 정보를 입력받으면 request.get_json()으로 url string을 받는 것을 가정했다.  
또한 `YoutubeAudioExtractor` 클래스와 내부 메서드인 extract_url_information이 실행되어 필요한 작업이 수행되도록 했다.  
이때 try-except문을 활용해서 예외 처리를 해주었는데, 이는 만약 요청받은 url이 유튜브에서 유효하지 않을 경우를 위함이다.  
가령 유효하지 않은 url이거나 일반 유저는 접근할 수 없는 url일 경우가 있는데, 이는 pytube에서 내부적으로 검증을 거쳐 Exception을 발행한다.  
이와 같은 경우는 Internal Server Error로 처리되어서는 안되므로 오류 사항을 담아 메인서버에 전달해서 오류 원인을 파악할 수 있게 돕는다.

```python
# utils.py

from pytube import YouTube
from werkzeug.exceptions import BadRequest
from datetime import timedelta, datetime

class YoutubeAudioExtractor:
    """Youtube 링크를 받으면 해당 링크에서 음원 추출
    """

    def __init__(self, urls):
        """유튜브 url 정보를 추출
            - url을 여럿 입력했을 경우 정규표현식을 통해 자동으로 리스트에 담아준다.

        Args:
            urls (string): get youtube urls
        """

        # 올바른 링크가 입력되지 않을 경우 빈 리스트가 생성될 수 있음
        self.urls_list = re.findall('http[a-zA-Z0-9:/.?=_\-]+', urls)

    def extract_url_information(self):
        """입력받은 Youtube urls의 영상 정보를 출력

        Returns:
            list: 각 url에 대한 정보가 dictionary 형태로 포함
        """

        results = []
        for url in self.urls_list:

            # NOTE: 사용할 수 없는 url의 경우 내부 raise 실행 (이유 제시)
            # NOTE: 사용 불가한 url이 포착된 경우 Main-Server에 오류 메시지 전달 구현 필요
            youtube = YouTube(url)

            results.append({
                'title': youtube.title,
                'thumbnail_url': youtube.thumbnail_url,
                'channel_id': youtube.channel_id,
                'length': str(timedelta(seconds=youtube.length)),
                'url': url
            })

        return results
```

`YoutubeAudioExtractor` 클래스를 구현한 내용이다. 초기화를 통해 해당 클래스가 urls 정보를 받으면 내부적으로 url에 해당하는 문자만 필터링하여 리스트에 담아둔다.  
이후 `extract_url_information` 메서드를 적용하면 각 url이 유튜브 영상에 해당하는지 내부적으로 점검한 뒤 관련 정보를 추출하여 리턴합니다.  
pytube의 YouTube 클래스에 url을 입력하면 자체적으로 유효한 url인지 확인하며, 그렇지 않을 경우 내부적으로 raise를 실행한다.

이렇게 유튜브 API를 간단하게 완성했다.

### Nest.js 메인 어플리케이션 서버

이제 메인 서버에서 API 통신하는 방법을 구현해보자
메인 서버는 nest.js 프로젝트로 생성하였다.

```typescript
// links.module.ts

import { Module } from '@nestjs/common';
import { LinksController } from './links.controller';
import { LinksService } from './links.service';
import { HttpModule } from '@nestjs/axios';

@Module({
  imports: [
    HttpModule.register({
      timeout: 30000,
      maxRedirects: 5,
    }),
  ],
  controllers: [LinksController],
  providers: [LinksService],
})
export class LinksModule {}
```

먼저 유튜브의 링크를 담당하는 links라는 이름의 모듈을 생성했다.  
외부 API와의 통신에서는 axios를 주로 활용하는 것을 확인할 수 있었다. 패키지는 @nestjs/axios와 axios를 설치한다.  
axios의 HttpModule을 import 하여 등록했고, 요청 응답을 기다려주는 시간, 최대 리디렉션 횟수를 임의로 지정해줬다.

```typescript
// links.controller.ts

import { Body, Controller, Post } from '@nestjs/common';
import { GetLinksInformationDto } from './dto/get-links-information.dto';
import { LinksService } from './links.service';
import { HttpService } from '@nestjs/axios';
import { catchError, map } from 'rxjs';

@Controller('links')
export class LinksController {
  constructor(private linksService: LinksService, private httpService: HttpService) {}

  @Post('/get-url-info')
  getLinksInformation(@Body() getLinksInformationDto: GetLinksInformationDto) {
    const res = this.httpService
      .post('http://127.0.0.1:5000/urls', getLinksInformationDto)
      .pipe(map((res) => res.data.error ?? res.data))
      .pipe(
        catchError((error) => {
          throw new Error(error);
        })
      );
    return res;
  }
}
```

컨트롤러에서는 link 경로의 get-url-info라는 엔드포인트를 생성하여 조회하고자 하는 유튜브 링크들을 string으로 나열하여 Body로 보내게 했다.  
Dto를 통해서 데이터의 검증을 진행하도록 했고, 코드 상에는 보이지 않지만 class-validator를 적용했다.  
여기서 주목할 점은 axios의 httpService와 rxjs가 함께 활용된다는 점이다.

```typescript
const res = this.httpService
  .post('http://127.0.0.1:5000/urls', getLinksInformationDto)
  .pipe(map((res) => res.data.error ?? res.data))
  .pipe(
    catchError((error) => {
      throw new Error(error);
    })
  );
```

axios를 활용한 API 요청을 보내는 과정을 간단하게 살펴보면,  
먼저 post 메서드로 요청 방식을 구분하고 있다. get 메서드도 가능하나 `getLinksInformationDto`와 같은 body 데이터 전달이 안되어 post로 설정했다.  
이후 pipe라는 메서드를 통해 응답 데이터를 받고, 만일 에러메시지를 받았을 경우 이를 출력하도록 한다.  
마지막으로 catchError를 통해 유튜브 API 서버에 원인을 알 수 없는 오류가 발생할 경우 Interval Server Error를 출력하도록 한다.  
만약 유튜브 링크 정보를 DB에 저장해야 한다면 컨트롤러가 아닌 서비스에서 TypeORM 레포지토리와 연동해서 저장할 수 있을 것이다.

### 테스트

Flask와 Nest.js 서버를 각각 켜서 아래와 같이 테스트 요청을 보낸다.
url은 띄어쓰기로 구분된 총 두 개의 링크를 한 줄에 보낸다. 정규표현식을 통해 아래 두 링크를 잘 구분하도록 처리했기 때문에 띄어쓰기만 잘 유지해 주면 두 링크에 대한 정보를 모두 받을 수 있다.

```zsh
curl -X POST "http://127.0.0.1:3000/links/get-url-info" \
     -H "Content-Type: application/json" \
     -d '{"urls": "https://youtu.be/DPROFkADGm0?si=0yoncWTQDOcY1344 https://youtu.be/bLgU-ZFPlfk?si=_m-LL0dHPXj5JBZi"}'
```

위 curl을 실행하면 아래와 같은 결과를 얻을 수 있다.

```javascript
[
  {
    channel_id: 'UCbsHYIZwk9pZIMp53ciDjOw',
    length: '10:15:06',
    thumbnail_url:
      'https://i.ytimg.com/vi/DPROFkADGm0/hq720.jpg?sqp=-oaymwEXCNUGEOADIAQqCwjVARCqCBh4INgESFo&rs=AOn4CLCenGmcMQpg25KEQsrBi2ot1RdW5w',
    title: 'Playlist | 디즈니 공주, 재즈',
    url: 'https://youtu.be/DPROFkADGm0?si=0yoncWTQDOcY1344',
  },
  {
    channel_id: 'UCVut4hqvrjQC4qDE3oc5qig',
    length: '2:30:00',
    thumbnail_url:
      'https://i.ytimg.com/vi/bLgU-ZFPlfk/hq720.jpg?sqp=-oaymwEXCNUGEOADIAQqCwjVARCqCBh4INgESFo&rs=AOn4CLDx_VzbzPPFlWSezd1Lhxe-6C9Cgw',
    title: '𝐏𝐥𝐚𝐲𝐥𝐢𝐬𝐭 나홀로 유럽거리에 빈티지 크리스마스 재즈 캐롤 𝐕𝐢𝐧𝐭𝐚𝐠𝐞 𝐂𝐡𝐫𝐢𝐬𝐭𝐦𝐚𝐬 𝐂𝐚𝐫𝐨𝐥 🎅🏻',
    url: 'https://youtu.be/bLgU-ZFPlfk?si=_m-LL0dHPXj5JBZi',
  },
];
```
