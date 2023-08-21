# Nest.js에서 S3 Bucket에 데이터 저장하기

Express와 마찬가지로 multer를 활용해서 업로드를 하면서 Nest.js답게 module, controller, service를 활용합니다.

```zsh
nest g mo uploads
nest g co uploads
nest g s uploads
```

위 CLI 명령어를 활용하면 uploads이름을 가진 폴더, 모듈, 컨트롤러, 서비스까지 기본 코드가 작성된 파일 생성이 가능합니다. nest.js에서는 task별로 구분해서 생성한 각각의 module.ts 파일을 app.module.ts에 전부 import 해야 하고, 모듈 내에서 컨트롤러와 서비스 등록 및 서비스 파일을 외부 모듈에 적용 시 export 해줘야 하는 등 놓치기 쉬운 설정이 많은데 CLI 명령어로 이들을 자동 생성하면 커넥션이 필요한 부분을 알아서 잘 이어주기 때문에 이로 인한 오류를 방지할 수 있습니다.

```typescript
// uploads.module.ts
import { Module } from '@nestjs/common';
import { UploadsController } from './uploads.controller';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { MulterModule } from '@nestjs/platform-express';
import { multerOptionsFactory } from './multer.options.factory';
import { UploadsService } from './uploads.service';

@Module({
  imports: [
    MulterModule.registerAsync({
      imports: [ConfigModule],
      useFactory: multerOptionsFactory,
      inject: [ConfigService],
    }),
  ],
  controllers: [UploadsController],
  providers: [UploadsService],
})
export class UploadsModule {}
```

먼저 uploads.module.ts 파일을 보면 imports로 `MulterModule`을 지정해주고 있는 것을 볼 수 있습니다. `ConfigModule`과 `ConfigService`를 MulterModule에 적용해주면 root 경로의 .env 파일에 AWS_ACCESS_KEY_ID와 AWS_SECRET_ACCESS_KEY, 버킷 정보, 리전 정보를 입력해두고서 가져와 사용할 수 있습니다.

참고로 민감 정보를 따로 관리하기 위핸 config 라이브러리는 `@nestjs/config` 설치 후 app.module.ts의 imports에 `ConfigModule.forRoot()`만 설정해두면 이후 부터는 .env에 든 변수를 process.env.변수명 형태로 어디든지 가져와 쓸 수 있게 됩니다.

위 MulterModule에서 중요한 부분은 useFactory에 적용된 함수 입니다. 해당 함수는 실제로 form-data 형태로 받은 파일을 가져와 버킷에 저장하는 역할을 합니다.

```typescript
// multer.options.factory.ts
import { S3Client } from '@aws-sdk/client-s3';
import { ConfigService } from '@nestjs/config';
import { MulterOptions } from '@nestjs/platform-express/multer/interfaces/multer-options.interface';
import * as multerS3 from 'multer-s3';
import { basename, extname } from 'path';

export const multerOptionsFactory = (configService: ConfigService): MulterOptions => {
  return {
    storage: multerS3({
      s3: new S3Client({
        region: configService.get('AWS_S3_REGION'),
        credentials: {
          accessKeyId: configService.get('AWS_ACCESS_KEY_ID'),
          secretAccessKey: configService.get('AWS_SECRET_ACCESS_KEY'),
        },
      }),
      bucket: configService.get('AWS_S3_BUCKET'),
      acl: 'public-read',
      contentType: multerS3.AUTO_CONTENT_TYPE,
      key(_req, file, callback) {
        const fileType = file.mimetype.split('/')[0];
        const ext = extname(file.originalname); // 확장자
        const baseName = basename(file.originalname, ext); // 확장자 제외
        const fileName =
          fileType === 'video'
            ? `videos/${baseName}-${Date.now()}${ext}`
            : `images/${baseName}-${Date.now()}${ext}`;
        callback(null, fileName);
      },
    }),
    // 파일 크기 제한
    /**
     * @note 이미지 파일과 동영상 파일 업로드 시 용량 제한을 분리하도록 하는 로직 필요
     */
    limits: {
      fileSize: 5 * 1024 * 1024, // 5mb
    },
  };
};
```

`multerOptionsFactory` 함수는 module.ts에서 userFactory에 적용되었던 함수입니다. 복잡해 보이지만 사실 MulterModule을 통해 S3등록에 필요한 정보를 등록하는 과정이며 크게 세 가지로 간단하게 구분할 수 있습니다.

1. s3 서비스 접속을 위한 정보 등록
2. s3에 생성해 둔 버킷을 지정
3. 데이터 저장 방식을 지정

storage 프로퍼티를 통해 어디에 지정할 지 결정하며 여기서는 MulterS3 클래스를 신규 생성하여 s3에 대한 정보를 등록하는 것을 볼 수 있습니다. 이로서 저장 위치는 s3 버킷이 됩니다.

추가적으로 limits 프로퍼티를 통해 업로드 가능 용량을 제한할 수 있습니다. 이 밖에도 `MulterOptions` 클래스 내부로 들어가면 옵션이 더 다양하게 있지만 현재로서는 이정도면 기본은 갖췄다고 볼 수 있을 것 같습니다.

Storage로 multerS3를 적용하고 있으며 위 코드에 기재된 옵션 외 옵션은 아래 문서에서 확인하실 수 있습니다.  
[multer-s3](https://www.npmjs.com/package/multer-s3)

코드를 자세히 보시면 이미지와 비디오 파일로 나눠서 각각의 폴더에 저장하도록 구현되어 있음을 알 수 있습니다. mimetype에는 전반부에 image인지 video인지 구분할 수 있는 정보를 주므로 이를 활용하여 두 타입을 나눠서 저장합니다.

```typescript
// uploads.controller.ts
import { Controller, Post, UploadedFile, UseInterceptors } from '@nestjs/common';
import { FileInterceptor } from '@nestjs/platform-express';
import { UploadsService } from './uploads.service';

@Controller('api/uploads')
export class UploadsController {
  constructor(private readonly uploadsService: UploadsService) {}

  @Post('profile-image')
  @UseInterceptors(FileInterceptor('file'))
  uploadProfileImage(@UploadedFile() file: Express.MulterS3.File) {
    // return this.uploadsService.uploadFile(file);
    return { url: file.location };
  }
}
```

이제 컨트롤러를 위와 같이 작성합니다. 위 코드에서 Interceptor라는 용어가 보입니다. 말 그대로 농구 공을 획! 가로치는 것과 같이 중간에 끼어들어 파일 프론트에서 보낸 파일 data에서 fieldName이 'file'로 된 정보를 가로채는 것과 같은 느낌을 줍니다.

FileInterceptor 함수에는 사실 옵셔널 파라미터가 존재하는데 해당 파라미터의 타입으로 `MulterOptions`이 지정되어 있습니다. 이는 multer.potions.factory.ts 파일을 통해 이미 정의가 되어 imports해 둔 상태이므로 MulterOptions의 s3로 타고 들어가 결론적으로 버킷에 파일 생성을 진행합니다. 마치 클라이언트와 백엔드 사이에 존재하는 일종의 미들웨어와 같은 느낌을 받을 수 있는데, Nest.js에는 미들웨어랑 인터셉터를 구분하고 있습니다.

공식 문서에 따르면 미들웨어의 경우 클라이언트에서 요청이 들어왔을 때 서버에 가기 전 특정 테스크를 수행하는 단방향 성질을 갖고 있지만 인터셉터의 경우 요청과 응답 과정에서 모두 클라이언트와 서버 가운데에서 개입이 가능한 것으로 보입니다.

그래서인지 FileInterceptor는 form-data 타입으로 파일이 요청과 함께 들어오면 이를 낚아 채어 버킷에 파일을 올려줍니다. (정식 요청이 마치지 않은 가운데 샛길로 새어 s3 서버에 응답을 보내줬습니다.)

이후 UploadedFile 데코레이터를 통해 업로드된 파일 정보에서 location 즉 파일이 저장된 url을 가져옵니다. 사실상 url 획득이 목적이었다면 service를 거칠 것도 없이 해당 값을 그대로 컨트롤러 단계에서 return해주면 됩니다. 하지만 추가적인 작업이 필요하다면 service로 연결할 수 있겠습니다.

```typescript
import { BadRequestException, Injectable } from '@nestjs/common';

@Injectable()
export class UploadsService {
  uploadFile(file: Express.MulterS3.File) {
    if (!file) {
      throw new BadRequestException('파일이 존재하지 않습니다.');
    }
    return { url: file.location };
  }
}
```

위 코드에서는 단순히 파일 존재 유무를 체크했을 뿐이지만 file 내부 프로퍼티를 대상으로 어떠한 작업을 가할 수도 있겠습니다.
