# Record - beta

## Overview

방송, 통신간 영상을 녹화, 녹음 하는 기능을 제공하고 있습니다.

### 지원범위

|  | Web | Android | iOS | Server |
| :--- | :--- | :--- | :--- | :--- |
| Communication - Video | X | X | X | X |
| Communication - Audio | O | O | O | X |
| Livecast - Video | X | X | X | O |
| Livecast - Audio | X | O | O | O |

### Client \(Beta\) - 방송/통신

안드로이드, iOS의 경우 녹음기능이 제공됩니다. 화면 녹화는 플렛폼에서 제공하는 전용 API를 사용할 수 있습니다. 녹음은 크게 다음과 같은 단계로 진행됩니다.

* 녹음 형식 설정
* 원본 소리 저장 \(dump\)
* 인코딩 \(unpack\)

녹음 기능의 특성상 원본을 로컬 저장소에 저장하게 되며 일정부분 저장소 용량을 차지하게 됩니다. 모든 저장이 완료 되면, 이를 압축하는 인코딩 과정이 필요하며, 이때는 CPU자원을 소모하게됩니다. 기기에 따라 하드웨어 가속등을 받지 못할 수도 있으며 클라이언트 자원을 소모함으로 필요한 경우에만 사용될 것을 권장합니다.

{% tabs %}
{% tab title="Web" %}
WebSDK의 경우 현재 녹음만을 지원합니다. 다음과 같이 config에 record관련 속성을 추가합니다.

```text
config = {credential, view,
    media: { audio:true, video:true,
        record: true,
        recordUrl: "https://blahblah.com/record/url"
        // recordUrl이 없을 경우 기본 리몬서버로 전송됨
        // recordUrl이 'local'인 경우 녹음만 하고 전송하지 않음.
    }
}
```

record 속성은 record를 enable하는 속성입니다. 만약 record만 true로 할경우 remotemonster의 별도의 서버에 녹음파일이 저장됩니다. recordUrl을 설정하면 당신이 원하는 서버로 녹음파일이 전달됩니다. SDK에서는 녹음 과정에 대하여 다음과 같은 이벤트를 제공합니다. 녹음파일은 webm 형식으로 전달됩니다.

```text
remon.onRecordEvent(event) {
    switch(event.event){
        case "stop: // 녹음이 종료되었을 때 발생합니다.
            console.log(event.id, event.size, event.file);
            break;
        case "upload": // 녹음 후 recordUrl로 전송이 시작할 때 발생합니다.
            console.log(event.id, event.size);
            break;
        case "progress": // 전송 중 파일이 큰 경우 업로드 도중에 호출될 수 있습니다.
            console.log(event.id, event.size);
            break;
        case "uploaded":// 전송이 완료되었을 때 발생합니다.
            console.log(event.id, event.size);
            break;
        case "error": //전송중 에러가 발생한 경우 발생합니다.
            console.log(event.id, event.size, error);
            break;
        }
    }
}
```
{% endtab %}

{% tab title="Android" %}
Remon SDK는 녹음만 지원됩니다. 영상의 녹화는 안드로이드의 `Mediaprojection API`를 사용하시면 됩니다. 녹음기능은 다음과 같이 사용 하시면 됩니다.

```java
remonCall = RemonCall.builder()
        .context(CallActivity.this)
        .localView(surfRendererLocal)
        .remoteView(surfRendererRemote)
        .serviceId(remonApplication.getConfig().getServiceId())
        .key(remonApplication.getConfig().getKey())
        .saveInputAudioToFile(true)                // 녹음기능 사용
        .fileSizeLimitBytes(300000000)             // 생성할 녹음파일의 최대사이즈
        .aecDumpFilePath(Environment.getExternalStorageDirectory().getPath() + File.separator + "Download/aecdump") // 파일 저장위
        .build();
```

 Builder에서 `saveInputAudioToFile`, `fileSizeLimitBytes`, `aecDumpFilePath` 등의 값을 지정해주시면 됩니다. 

녹음 파일은 별도의 포맷으로 저장 되므로,  Unpack 작업이 필요합니다. 

```java
String source = Environment.getExternalStorageDirectory().getPath() + File.separator + "Download/aecdump";
String result = Environment.getExternalStorageDirectory().getPath() + File.separator + "Download/record.wav";
UnpackAecdump unpackAecdump = new UnpackAecdump();
try {
    unpackAecdump.run(RouterActivity.this, source , result, state -> {
        switch (state) {
            case UNPACK_DONE:
                break;
            case WRITE_DONE:
                runOnUiThread(() -> Toast.makeText(RouterActivity.this, "녹음파일 저장 완료.", Toast.LENGTH_SHORT).show());
                break;
            case ERROR:
                runOnUiThread(() -> Toast.makeText(RouterActivity.this, "잘못된 녹음 파일입니다.", Toast.LENGTH_SHORT).show());
                break;
        }
    });
} catch (Exception e) {
    e.printStackTrace();
}
```

 UnpackAecDump의 run\(\)함수를 통해 Upack작업을 수행할 수 있으며, 인자는 다음과 같습니다 run\(`context`, `dump파일위치`, `음성파일저장위치`, `OnRecordAudioStatusListener`\) 

`OnRecordAudioStatusListener` 의 state로 진행과정을 확인할 수 있습니다. UNPACK\_DONE은 upack완료, WRITE\_DONE은 wav파일 생성 완료 입니다.
{% endtab %}

{% tab title="iOS - Swift" %}
```swift
remonCall.onComplete { () in            
    //덤프 기록 시작    
    self.remonCall.startDump(
        withFileName: "audio.aecdump", maxSizeInBytes: 100 * 1024 * 1024) //100mb
}

remonCall.onClose {
    //덤프 기록 중지
    self.remonCall.stopDump()
    
    // m4a 형식으로 변환을 원하는 경우 변환작업 시작
    // 이 과정은 많은 시간이 소요 됩니다.
    RemonClient.unpackAecDump( dumpName: "audio.aecdump", 
                               resultFileName: "unpack.m4a", 
                               progress: {
        (error, state) in
        if let error = error {
            // 에러가 발생 하였습니다. 프로그래스 종료.
        } else {
            // 디코딩 상태 
            /*
            case START = 0
            case WARNING
            case ERROR // 이 경우 error가 존재 합니다. 프로그래스 종료.
            case WROTE
            case COMPLTE  //프로그래스 종료.
            */
        }
    })
}
```
{% endtab %}

{% tab title="iOS - ObjC" %}
```objectivec
[self.remonCall onCompleteWithBlock:^{
    //덤프 기록 시작
    [self.remonCall startDumpWithFileName:
        @"audio.aecdump" maxSizeInBytes:100 * 1024];
}];

[self.remonCall onCloseWithBlock:^{
    //덤프 기록 중지
    [self.remonCall stopDump];
    
    // m4a 형식으로 변환을 원하는 경우 변환작업 시작
    // 이 과정은 많은 시간이 소요 됩니다.
    [RemonClient unpackAecDumpWithDumpName:
        @"audio.aecdump" resultFileName:@"unpack.m4a" progress:
        ^(NSError * _Nullable erro, enum REMON_AECUNPACK_STATE state) {
            // 디코딩 상태
        }
    ];
}];
```
{% endtab %}
{% endtabs %}

이외에도 각 플렛폼은 자체적으로 녹화가능한 기능을 제공하고 있습니다. 아래를 참고하여 클라이언트에서 독자적으로 녹화 기능을 제작할수 있습니다. 아래 서버녹화와 다르게 이기능을 활용하면 사용자가 보는 모든 화면을 녹화할 수 있습니다.

#### Web Media Element Recording API

{% embed url="https://developer.mozilla.org/en-US/docs/Web/API/MediaStream\_Recording\_API/Recording\_a\_media\_element" %}

#### Android Screen Recording API

{% embed url="https://developer.android.com/reference/android/media/projection/MediaProjection" %}

#### iOS Screen Recording API

{% embed url="https://developer.apple.com/documentation/replaykit" %}

### Server \(Beta\) - 방송

방송기능 사용시 서버를 통해 영상 녹화 기능이 제공됩니다. 이를 통해 녹화를 하면, 최종 사용자가 보는 UI가 조합된 화면이 아닌, 송출하는 원본영상을 녹화 받을 수 있습니다. 방송이 진행되면 서버에서 녹화가 되며 방송이 종료되어 녹화가 완료된 순간, 사용자가 지정한 주소로 Webhook이 호출됩니다. 이 전문안의 규격을 갖고 사용자는 영상을 다운로드 받을 수 있습니다. 영상은 6시간 후 리모트몬스터 서버에서 삭제됩니다.

서버를 통한 영상녹화 기능은 현재 베타기능이며 향후 별도의 과금체계와 함께 공식지원될 예정입니다.

#### Setting Webhook URL

REST POST가 호출될 URL 주소를 Admin API를 통해 설정 합니다. 아래를 참고하여 설정을 합니다.

리모트몬스터에서 해당 설정이 완료되면 지정된 주소로 POST가 호출됩니다. 자세한 내용은 아래를 참고하세요.

{% page-ref page="../api/webhook-livecast.md" %}

thumbnail를 얻길 원하면 아래와 같은 외부 API 등을 검토해 보세요.

{% embed url="https://cloud.google.com/video-intelligence/" %}

{% embed url="https://aws.amazon.com/ko/rekognition/" %}

{% embed url="https://azure.microsoft.com/ko-kr/services/media-services/video-indexer/" %}

#### Downloading Media

제공받은 URL에 접근함으로써 녹화된 영상을 다운로드 받을 수있습니다. 영상은 정상적으로  Webhook이 호출 된 이후 6시간 후 삭제되며 그 이전에는 언제든지 다운로드가 가능합니다.

간단하게 아래와 같이 `curl`, `wget`으로 테스트가 가능하며 이 기능을 서비스의 서버에서 구현하여 사용할 수 있습니다.

```text
curl -O https://s3.amazonaws.com/__SERVICEID__/__RAND__/__CHANNELID__.mp4
wget https://s3.amazonaws.com/__SERVICEID__/__RAND__/__CHANNELID__.mp4
```

#### Simple Webhook Server - by hooka

간단하게 서버를 만들어서 웹훅을 확인하고 녹화를 다운로드 할 수 있습니다. 아래와 같이 docker 컨테이너와 간단한 설정을 통해 다운로드 서버를 구축 가능합니다.

```bash
$ docker run -v ./webhooks.json:/src/webhooks.json -p 3000:3000 danistefanovic/hooka
```

{% code title="webhooks.json" %}
```javascript
[
    {
        "method": "POST",
        "path": "/MYSLUG",
        "command": "wget $URL;",
        "parseJson": [
            {
                "query": "payload.id",
                "variable": "ID"
            },
            {
                "query": "payload.url",
                "variable": "URL"
            }
        ]
    }
]
```
{% endcode %}

보다 자세한 사용법은 아래를 확인하세요.

{% embed url="https://github.com/danistefanovic/hooka" %}

#### Simple Webhook Server - by nodejs

혹은 아래와 같이 간단한 nodejs 서버 프로그래밍으로 웹훅을 수신 가능합니다.

```javascript
const express = require('express')
const bodyParser = require('body-parser')
const app = express()

app.use(bodyParser.urlencoded())
app.use(bodyParser.json())

app.post('/__MYSLUG__', (req, res) => {
    console.log(JSON.stringify(req.body))
    res.end()
})

const server = app.listen(3000, '127.0.0.1', () => {
    const host = server.address().address
    const port = server.address().port
    console.log("Example app listening at http://%s:%s", host, port)
})
```

