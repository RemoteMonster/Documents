# Communication

## 기본 설정 <a id="undefined"></a>

통신을 하기 전에 프로젝트 설정을 진행 합니다.​

## 개발 <a id="undefined-1"></a>

통신을 기능은 이용하기 위해서는 `RemonCall` 클래스를 이용합니다. `RemonCall`클래스의 `connect()` 함수를 이용하여 채널 생성 및 접속이 가능합니다.

전체적인 구성과 흐름은 아래를 참고하세요.​​

{% page-ref page="../overview/flow.md" %}

{% page-ref page="../overview/structure.md" %}

### View 등록

통화중 스스로의 모습을 보거나 상대방의 모습을 보기위한 뷰가 필요합니다. 자기 자신의 모습은 Local View, 상대방의 모습은 Remote View로 등록을 합니다.

{% tabs %}
{% tab title="Web" %}
```javascript
<!-- local view -->
<video id="localVideo" autoplay muted></video>
<!-- remote view -->
<video id="remoteVideo" autoplay></video>
```
{% endtab %}

{% tab title="Android" %}
```markup
<!-- local view -->
<com.remotemonster.sdk.PercentFrameLayout
    android:id="@+id/perFrameLocal"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <org.webrtc.SurfaceViewRenderer
        android:id="@+id/surfRendererLocal"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
</com.remotemonster.sdk.PercentFrameLayout>
```

```markup
<!-- remote view -->
<com.remotemonster.sdk.PercentFrameLayout
    android:id="@+id/perFrameRemote"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <org.webrtc.SurfaceViewRenderer
        android:id="@+id/surfRendererRemote"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
</com.remotemonster.sdk.PercentFrameLayout>
```
{% endtab %}

{% tab title="iOS - Swift" %}
Interface Builder를 통해 지정 하게 되며 iOS - Getting Start에 따라 환경설정을 했다면 이미 View등록이 완료된 상태 입니다. 혹, 아직 완료가 안된 상태라면 아래를 참고하세요.

{% page-ref page="../ios/ios-getting-started.md" %}
{% endtab %}

{% tab title="iOS - ObjC" %}
Interface Builder를 통해 지정 하게 되며 iOS - Getting Start에 따라 환경설정을 했다면 이미 View등록이 완료된 상태 입니다. 혹, 아직 완료가 안된 상태라면 아래를 참고하세요.

{% page-ref page="../ios/ios-getting-started.md" %}
{% endtab %}
{% endtabs %}

보다 더 자세한 내용은 아래를 참고하세요.

{% page-ref page="../web/web-view.md" %}

{% page-ref page="../android/android-media.md" %}

{% page-ref page="../ios/ios-media.md" %}

### 통화 걸기

`connect()` 함수에 전달한 `channelId` 값에 해당하는 채널이 존재하지 않으면 채널이 생성되고, 다른 사용자가 해당 채널에 연결 하기를 대기 하는 상태가 됩니다. 이때 해당 `channelId`로 다른 사용자가 연결을 시도 하면 연결이 완료 되고, 통신이 시작 됩니다.

{% tabs %}
{% tab title="Web" %}
```javascript
// <video id="localVideo" autoplay muted></video>
// <video id="remoteVideo" autoplay></video>
let myChid
​
const config = {
  credential: {
    serviceId: 'MY_SERVICE_ID',
    key: 'MY_SERVICE_KEY'
  },
  view: {
    local: '#localVideo',
    remote: '#remoteVideo'
  }
}
​
const listener = {
  onConnect(channelId) {
    myChannelId = channelId
  },
  onComplete() {
    // Do something
  }
}
​
const caller = new Remon({ listener, config })
caller.connectCall()
```
{% endtab %}

{% tab title="Android - Java" %}
```java
caller = RemonCall.builder()
    .serviceId("MY_SERVICE_ID")
    .key("MY_SERVICE_KEY")
    .context(CallActivity.this)
    .localView(surfRendererLocal)
    .remoteView(surfRendererRemote)
    .build();
​
caller.onConnect((channelId) -> {
    myChannelId = channelId  // Callee need chid from Caller for connect
});
​
caller.onComplete(() -> {
    // Caller-Callee connect each other. Do something
});

caller.connect("CHANNEL_NAME");
```
{% endtab %}

{% tab title="Android - Kotlin" %}
```kotlin
caller = RemonCall.builder()
    .serviceId("MY_SERVICE_ID")
    .key("MY_SERVICE_KEY")
    .context(CallActivity.this)
    .localView(surfRendererLocal)
    .remoteView(surfRendererRemote)
    .build()
​
caller.onConnect { channelId -> 
    myChannelId = channelId  // Callee need chid from Caller for connect
}
​
caller.onComplete {
    // Caller-Callee connect each other. Do something
}

caller.connect("CHANNEL_NAME")
```
{% endtab %}

{% tab title="iOS - Swift" %}
```swift
let caller = RemonCall()

caller.onConnect { [weak self](channelId) in
    let myChannelId = channelId          // Callee need channelId from Caller for connect
}

caller.connect("MY_CHANNEL_ID")
```
{% endtab %}

{% tab title="iOS - ObjC" %}
```objectivec
RemonCall *caller = [[RemonCall alloc] init];
​
[caller onConnectWithBlock:^(NSString * _Nullable channelId) {
// Callee need channelId from Caller for connect
    [self setMyChannelId:channelId];
}];
​
[caller connect:chId :@"MY_CHANNEL_ID"];
```
{% endtab %}
{% endtabs %}

### 통화 받기 <a id="undefined-3"></a>

`connect()` 함수에 접속을 원하는 `channelId`값을 넣습니다. 대기상태에 있던 사용자와 연결을 진행하고, 정상 연결이 완료되면 onComplete 콜백이 호출됩니다.

{% tabs %}
{% tab title="Web" %}
```javascript
// <video id="localVideo" autoplay muted></video>
// <video id="remoteVideo" autoplay></video>
const config = {
  credential: {
    serviceId: 'MY_SERVICE_ID',
    key: 'MY_SERVICE_KEY'
  },
  view: {
    local: '#localVideo',
    remote: '#remoteVideo'
  }
}
​
const listener = {
  onComplete() {
    // Do something
  }
}
​
const callee = new Remon({ listener, config })
callee.connectCall('MY_CHANNEL_ID')
```
{% endtab %}

{% tab title="Android - Java" %}
```java
callee = RemonCall.builder()
    .serviceId("MY_SERVICE_ID")
    .key("MY_SERVICE_KEY")
    .context(CallActivity.this)
    .localView(surfRendererLocal)
    .remoteView(surfRendererRemote)
    .build();

callee.onComplete(() -> {
    // Caller-Callee connect each other. Do something
});

callee.connect("MY_CHANNEL_ID");
```
{% endtab %}

{% tab title="Android - Kotlin" %}
```kotlin
callee = RemonCall.builder()
    .serviceId("MY_SERVICE_ID")
    .key("MY_SERVICE_KEY")
    .context(CallActivity.this)
    .localView(surfRendererLocal)
    .remoteView(surfRendererRemote)
    .build()

callee.onComplete {
    // Caller-Callee connect each other. Do something
}

callee.connect("MY_CHANNEL_ID")
```
{% endtab %}

{% tab title="iOS - Swift" %}
```swift
let callee = RemonCall()

callee.onComplete {
    // Caller-Callee connect each other. Do something
}

callee.connect("MY_CHANNEL_ID")
```
{% endtab %}

{% tab title="iOS - ObjC" %}
```objectivec
RemonCall *callee = [[RemonCall alloc] init];
​
[callee onCompleteWithBlock:^{
    // Caller-Callee connect each other. Do something
}];
​
[callee connect:chId :@"MY_CHANNEL_ID"];
```
{% endtab %}
{% endtabs %}

### Callbacks <a id="observer"></a>

개발중 다양한 상태 추적을 돕기 위한 Callback을 제공 합니다. 

* 안드로이드 2.4.13, iOS 2.6.9 버전부터 콜백은 모두 UI Thread 에서 호출됩니다.

{% tabs %}
{% tab title="Web" %}
```javascript
const listener = {
  onInit(token) {
    // UI 처리등 remon이 초기화 되었을 때 처리하여야 할 작업
  },
​  
  onConnect(channelId) {
    // 통화 생성 후 대기 혹은 응답
  },
​
  onComplete() {
    // Caller, Callee간 통화 시작
  },
​  
  onClose() {
    // 종료
  }
}
```
{% endtab %}

{% tab title="Android - Java" %}
```java
remonCall = RemonCall.builder().build();

remonCall.onInit(() -> {
    // UI 처리등 remon이 초기화 되었을 때 처리하여야 할 작업
});
​
remonCall.onConnect((channelId) -> {
    // 통화 생성 후 대기 혹은 응답
});
​
remonCall.onComplete(() -> {
    // Caller, Callee간 통화 시작
});
​
remonCall.onClose(() -> {
    // 종료
});
```
{% endtab %}

{% tab title="Android - Kotlin" %}
```kotlin
remonCall = RemonCall.builder().build()

remonCall.onInit {
    // UI 처리등 remon이 초기화 되었을 때 처리하여야 할 작업
}
​
remonCall.onConnect { channelId ->
    // 통화 생성 후 대기 혹은 응답
}
​
remonCall.onComplete {
    // Caller, Callee간 통화 시작
}
​
remonCall.onClose {
    // 종료
}
```
{% endtab %}

{% tab title="iOS - Swift" %}
```swift
let remonCall = RemonCall()

remonCall.onInit { [weak self](token) in
    // UI 처리등 remon이 초기화 되었을 때 처리하여야 할 작업
}
​
remonCall.onConnect { [weak self](channelId) in
    // 해당 'chid'로 미리 생성된 채널이 없다면 다른 사용자가 해당 'chid'로 연결을 시도 할때 까지 대기 상태가 됩니다. 
}
​
remonCall.onComplete { [weak self] in
    // Caller, Callee간 통화 시작
}
​
remonCast.onClose { [weak self](closeType) in
    // 종료
}
```
{% endtab %}

{% tab title="iOS - ObjC" %}
```objectivec
RemonCall *remonCall = [[RemonCall alloc] init];

[remonCall onInitWithBlock:^{
    // Things to do when remon is initialized, such as UI processing, etc.
}];

[remonCallter onConnectWithBlock:^(NSString * _Nullable chId) {
    // Make a call then wait the callee
}];

[remonCall onCompleteWithBlock:^{
    // Start between Caller and Callee
}];

[remonCall onCloseWithBlock:^{
    // End calling
}];
```
{% endtab %}
{% endtabs %}

더 많은 내용은 아래를 참조 하세요.​

{% page-ref page="callbacks.md" %}

### Channel <a id="channels"></a>

랜덤채팅등과 같은 서비스에서는 전체 채널 목록을 필요로 하게 됩니다. 이를 위한 전체 채널 목록을 제공합니다.

{% tabs %}
{% tab title="Web" %}
```javascript
const remonCall = new Remon()
const calls = await remonCall.fetchCalls()
```
{% endtab %}

{% tab title="Android - Java" %}
```java
remonCall = RemonCall.builder().build();

remonCall.fetchCalls();
remonCall.onFetch( calls -> {
    // Do something
});
```
{% endtab %}

{% tab title="Android - Kotlin" %}
```kotlin
remonCall = RemonCall.builder().build()

remonCall.fetchCalls()
remonCall.onFetch { calls -> 
    // Do something
}
```
{% endtab %}

{% tab title="iOS - Swift" %}
```swift
let remonCall = RemonCall()

remonCall.fetchCalls { (error, results) in
    // Do something
}
```
{% endtab %}

{% tab title="iOS - ObjC" %}
```objectivec
RemonCall *remonCall = [[RemonCall alloc]init];
[remonCall fetchCastsWithIsTest:YES
              complete:^(NSArray<RemonSearchResult *> * _Nullable chs) {
                            // Do something
}];
```
{% endtab %}
{% endtabs %}

채널에 대한 더 자세한 내용은 아래를 참고하세요.​

{% page-ref page="channel.md" %}

### 종료 <a id="undefined-4"></a>

모든 통신이 끝났을 경우 꼭 RemonCast객체를 `close()`해주어야 합니다. close를 통해서 모든 통신자원과 미디어 스트림 자원이 해제됩니다.

{% tabs %}
{% tab title="Web" %}
```javascript
const remonCall = new Remon()
remonCall.close()
```
{% endtab %}

{% tab title="Android - Java" %}
```java
remonCall = RemonCall.builder().build();
remonCall.close();
```
{% endtab %}

{% tab title="Android - Kotlin" %}
```kotlin
remonCall = RemonCall.builder().build()
remonCall.close()
```
{% endtab %}

{% tab title="iOS - Swift" %}
```swift
let remonCall = RemonCall()
remonCall.closeRemon()
```
{% endtab %}

{% tab title="iOS - ObjC" %}
```objectivec
RemonCall *remonCall = [[RemonCall alloc]init];
[remonCall closeRemon];
```
{% endtab %}
{% endtabs %}

### 기타 <a id="undefined-5"></a>

아래를 통해 보다 자세한 설정,  실 서비스를 위한 프렉티스등 다양한 내용을 확인해 보세요.

{% page-ref page="config.md" %}

{% page-ref page="network-environment.md" %}

