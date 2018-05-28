# Android - Getting Start

## 준비사항

* 안드로이드 개발 환경
* minSdkVersion 18 이상
* java 1.8 이상

## 프로젝트 생성 및 설정

### 프로젝트 생성 및 API 레벨 설정

API Level 18이상으로 설정 합니다.

![](../.gitbook/assets/image.png)

### Compatibility 설정 

Open Module Settings에서 Source Compatibility, Target Compatibility를 1.8 이상으로 설정해줍니다.

![](../.gitbook/assets/image%20%284%29.png)

### Module Gradle 설정

build.gradle\(Module:app\) 의 dependencies에 remon sdk를 추가합니다.

```groovy
dependencies {
    ...
    
    /* Remote monster WebRTC library */
    compile 'com.remotemonster.sdk:2.0.4'
}
```

### Project Gradle 설정

build.gradle\(project:name\)의 allproject에 다음 주소를 입력합니다.

```groovy
allprojects {
    repositories {
        google()
        jcenter()
        maven { url 'https://demo.remotemonster.com/artifactory/libs-release-local' }
    }
}
```

이제 안드로이드 스튜디오를 동기화하면 자동으로 Remote Monster의 Android SDK를 다운로드 받게 됩니다.

### Permission 설정

안드로이드 최신 버전의 경우 앱의 권한에 대해 처음 앱 사용시 사용자에게 직접 묻게 됩니다. 처리해야할 권한은 다음과 같습니다.

```java
public static final String[] MANDATORY_PERMISSIONS = {
  "android.permission.INTERNET",
  "android.permission.CAMERA",
  "android.permission.RECORD_AUDIO",
  "android.permission.MODIFY_AUDIO_SETTINGS",
  "android.permission.ACCESS_NETWORK_STATE",
  "android.permission.CHANGE_WIFI_STATE",
  "android.permission.ACCESS_WIFI_STATE",
  "android.permission.READ_PHONE_STATE",
  "android.permission.BLUETOOTH",
  "android.permission.BLUETOOTH_ADMIN",
  "android.permission.WRITE_EXTERNAL_STORAGE"
};
```

## 개발

이제 모든 준비가 끝났습니다. 아래를 통해 세부적인 개발 방법을 확인하세요.

### 방송

{% page-ref page="android-communication.md" %}

### 통신

{% page-ref page="android-livecast.md" %}


