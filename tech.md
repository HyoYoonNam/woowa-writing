## 0. 이 글을 쓴 이유

총대마켓 안드로이드 앱을 개발하면서 카카오 소셜 로그인 기능을 구현해야 했다. 소셜 로그인기능은 처음 해 보는 것이고, 구현을 해 보기 전까지는 어떻게 구현되는지 감이 잡히지 않았다. 하지만 소셜 로그인 플로우에 대해 이해해 보고 싶었고, 우리의 서비스가 어떤 방식으로 유저를 검증하고, 유저 정보를 관리하는지를 알고 싶었다. 그래서 이 파트를 내가 해보고 싶다고 팀원들에게 말했고, 내가 맡게 되었다! 하지만 내 예상대로 많은 어려움이 기다리고 있었다.. 시간과 노력을 많이 들인 부분이라, 내가 겪었던 난관들과 새로 알게 된 내용들을 기록해 두고자 한다! 😄

## 1. 왜 소셜 로그인을 쓸까?

어떤 서비스에 로그인하는 방법은 크게 두 가지로 나뉠 것 같다. 첫 번째는 아이디와 비밀번호를 입력해 로그인하는 방법. 두 번째는 카카오, 네이버, 구글과 같은 소셜 로그인을 사용하는 방법이다. 소셜 네트워크 서비스가 활성화되기 이전에는 대부분 전자의 방법을 사용했지만, 최근에는 후자의 방법인 소셜 로그인을 사용하는 서비스가 늘고 있다. 왜일까?

아이디와 비밀번호로 회원가입해 서비스에 회원정보를 넘겨주고 로그인하는 방식은 크게 두 가지 문제점이 있다.

1.  사용자 입장에서 불편하다. 아이디와 비밀번호를 기억해 두었다가 매 로그인 시마다 새로 입력해야 한다.
2.  사용자가 해당 서비스의 보안을 신용하기 어렵다.

1번이야 단순히 불편한 정도에 그치지만, 2번의 경우 치명적인 문제를 초래할 수 있다. 개인이나 학생들이 프로젝트로 개발한 소규모 서비스가 이에 해당한다. 이들은 서비스 운영진에 보안 전문가도 없고, 보안을 강화하지도 않았을 텐데 사용자들이 믿고 자신의 개인 정보를 넘겨줄 수 있겠는가?! 

좀 더 자세히 설명하자면, 만약 유저의 아이디와 비밀번호를 총대마켓 서버가 직접 관리한다면 외부에서의 공격에 취약하다. 외부의 해커가 총대마켓 서버를 해킹한다면 유저의 아이디, 비밀번호를 탈취할 수 있기 때문이다. 하지만 카카오 소셜 로그인을 사용한다면 아이디와 비밀번호를 총대마켓 서버가 직접 관리하지 않고, 카카오 계정을 사용하기 때문에 해커가 아이디와 비밀번호를 알아내려면 총대마켓이 아닌 카카오 서버를 해킹해야 한다. 이는 총대마켓 서버를 해킹하는 것 보다 훨씬 어려운 일이기 때문에 보안 상 더욱 안전하다.

이러한 이유들로 인해 대다수의 프로젝트는 **OAuth**를 사용하게 된다!


> OAuth란?  
> OAuth는 인터넷 사용자들이 비밀번호를 제공하지 않고 다른 웹사이트 상의 자신들의 정보에 대해 웹사이트나 애플리케이션의 접근 권한을 부여할 수 있는 공통적인 수단으로서 사용되는, 접근 위임을 위한 개방형 표준이다.

쉽게 말해 개발자들이 운영하는 서비스의 보안을 자신들이 관리하지 않고 믿을 수 있는 타 서비스에 맡기는 것이다.
<img src="https://github.com/user-attachments/assets/29a351de-6639-4225-a60e-8e4ed94b8d1c" width=400 />


한국인 대다수가 이미 사용하고 있는 카카오, 네이버 등의 서비스가 OAuth에 많이들 사용된다. 이미 대다수의 한국인이 카카오톡을 이용하고 있기 때문에, 거의 모든 유저를 커버할 수 있겠다는 판단 하에 우리 총대마켓은 카카오 소셜 로그인을 구현하기로 결정하였다 😁

## 2. 소셜 로그인 과정 이해하기

카카오 소셜 로그인을 사용하기로 했으니, 카카오를 통해 총대마켓에 회원가입하고 로그인 하는 과정을 이해할 필요가 있었다. 나는 지금까지 어떤 서비스에서 카카오 소셜 로그인을 할 때, 그냥 카카오 아이디와 비밀번호로 자동으로 회원가입 해주고 그걸로 로그인되는 건가보다! 생각했는데 그렇지 않았다..

소셜 로그인 플로우를 이해하기 위해서 많은 블로그를 참고하며 공부해 보았다. 물론 처음에는 블로그 글을 읽어봐도 이해가 되지 않았지만, 직접 따라해 보다 보니 조금씩 이해가 되었다! 일반적으로 카카오 소셜 로그인은 다음과 같이 이루어진다.

1.  클라이언트가 카카오 계정에 로그인 해서 카카오 Access Token을 발급받는다. 이 과정에서 유저가 개인정보 제공 동의를 한 경우, 카카오는 사용자의 기본 정보(닉네임, 성별, 이메일 등)를 포함한 정보를 제공하고, 이를 총대마켓에서 사용할 수 있다.
2.  클라이언트는 발급받은 카카오 Access Token을 총대마켓 서버로 보낸다.
3.  총대마켓 서버는 카카오 Access Token으로 카카오 서버와 통신하고, 카카오 서버로부터 유저 정보를 내려받는다.
4.  총대마켓 서버는 카카오 서버로부터 받은 유저 정보에 해당하는 총대마켓 계정을 생성하고, 해당 유저의 총대마켓 Access Token을 생성한다. 이 총대마켓 Access Token을 클라이언트로 보낸다. 여기까지 성공적으로 완료되었다면 로그인이 성공한다.
5.  이후 클라이언트는 서버로부터 받은 총대마켓 Access Token을 모든 서버 통신 시마다 보내며, 서버는 이 Access Token을 받아 현재 총대마켓에 접속한 유저가 누구인지 검증한다.

여기서 주의해야 할 점으로는 카카오 Access Token과 총대마켓 Access Token이 다르다는 것이다!! 😲 (굉장히 헷갈렸던 부분..) 

카카오 Access Token은 카카오 서버와 통신하는 데 사용되고, 총대마켓 Access Token은 총대마켓 서버와 통신하는 데 사용된다. 즉 카카오 Access Token은 총대마켓과 클라이언트 사이의 검증 작업에는 쓰이지 않는다. 현재 접속한 유저가 누구인지 검증하는 것은 총대마켓에서 생성한 Access Token으로 이루어진다.

또 보안을 위해 총대마켓의 Access Token은 유효기간이 30분으로 짧게 설정되어 있다. 즉 유저는 총대마켓에 로그인한 이후 30분이 지나면 다시 로그인을 해야 하는 불편함이 발생하게 된다.. 이 문제를 해결하기 위해 Refresh Token이라는 것이 존재한다.

Refresh Token은 클라이언트가 갖고 있다가 Access Token이 만료되었을 시 서버로 보낸다. 그럼 서버는 Refresh Token을 검증한 후 해당 유저가 누구인지 알아내고, 해당 유저의 Access Token을 재발급해준다. 그러면 재발급받은 Access Token으로 다시 서버와 통신하면 된다! 👍

써 놓고 보니 꽤나 복잡하다.. 이해하는 데 엄청 오래 걸렸다.

## 3. 카카오 소셜 로그인 구현하기

이제 내가 실제로 카카오 소셜 로그인을 구현한 방법을 적어보겠다!

[Kakao Developers](https://developers.kakao.com/docs/latest/ko/kakaologin/android)

위의 카카오 로그인 공식문서를 보며 따라했다. 하지만 공식문서를 보고도 이해가 되지 않는 부분이 많아, 다른 블로그의 포스팅이나 유튜브 영상도 많이 참고했다.

크게 아래의 두 가지 단계로 구현할 수 있다.

1.  카카오 개발자 콘솔(kakao developers)에서 앱 등록하기
2.  우리 앱에서 kakao SDK 사용하기

### 3-1. kakao developers에서 앱 등록하기

먼저 카카오 소셜 로그인을 사용하기 위해 kakao developers에서 등록해 주어야 하는 것들이 있었다. 이 과정은 카카오 SDK를 사용하기 위한 사전 준비를 하고, 권한을 부여받는 작업이다. 아래 링크에서 진행해 주면 된다! 😊

[Kakao Developers](https://developers.kakao.com/)


#### 1) 앱 등록

<img src=https://github.com/user-attachments/assets/2629d471-3026-44ca-b7d1-ac31dab3a226 width=400 /> <br>
<img src="https://github.com/user-attachments/assets/13a24271-7517-472b-b6f6-4b7c150cd367" width=400 />


**내 어플리케이션** 탭에 들어가 우리 앱의 정보를 등록해 준다.

#### 2) 카카오 네이티브 앱 키 발급

<img src=https://github.com/user-attachments/assets/a46e3391-7351-4318-bdc1-44591e65f44f width=600 />

앱을 등록하게 되면 **앱 설정 > 앱키** 탭에서 네이티브 앱 키를 발급받을 수 있다. 이 키를 사용해 카카오 SDK를 초기화시켜주어야 한다. (주의! 이 키는 private 하게 관리해야 하기 때문에 local.properties에서 사용하고 git ignore 해야 한다. 나는 처음에 이걸 Git 원격 저장소에 올리는 실수를 했다는.. 하하 테스트용 앱이어서 다행이었지..😅)

카카오 SDK는 카카오의 개발자들이 카카오의 다양한 서비스와 기능을 우리 앱에서 쉽게 사용할 수 있도록 만들어 놓은 도구 모음이다. 이를 이용해 소셜 로그인을 비롯한 카카오 API를 우리의 앱에서 쉽게 사용할 수 있는 것이다!

#### 3) 키 해시 등록

완료되었다면 **앱 설정 > 플랫폼** 탭에 들어가 키 해시를 등록해 주어야 한다.

<img src=https://github.com/user-attachments/assets/a8ae58df-2259-457f-954a-3744f264139a width=600 />

이 **키 해시**는 앱이 카카오 SDK를 사용해 카카오 API를 호출할 때, 해당 앱을 식별하는 데 사용된다. 즉 키 해시가 등록된 앱만 카카오 SDK를 사용할 수 있다.

키 해시는 팀원들 각각의 안드로이드 스튜디오 프로젝트마다 다른 값을 가지고 있고, 등록한 사람의 프로젝트에서 빌드한 앱에서만 카카오 로그인을 할 수 있다. 따라서 모든 팀원이 키 해시를 추출해서 이곳에 등록해주어야 했다. (그런데 사실 지금은 키 해시가 무엇인지 알아보아도 잘 이해가 가지 않아서.. 나중에 자세히 공부해 보려고 한다!!)

#### 4) 개인정보 동의항목 설정 (선택)

카카오 소셜 로그인 시 카카오 Access Token 뿐만 아니라 추가로 유저의 개인정보를 사용하고 싶을 때가 있다. 이런 경우 **제품 설정 > 카카오 로그인 > 동의 항목**에 들어가면 원하는 개인정보를 동의받는 설정을 할 수 있다.

<img src=https://github.com/user-attachments/assets/cba4977a-b2a0-4c83-bb91-d66c31412903 width=600 /> <br>
<img src=https://github.com/user-attachments/assets/921a2c73-e358-4ddc-943d-df3d652a9e87 width=300 />

위와 같은 화면 본 적 있을 것이다! 이 곳에서 사용자가 자신의 개인정보 제공에 동의하면, 우리 총대마켓에서 해당 개인정보를 사용할 수 있게 된다.

사용자가 카카오에 가입한 개인정보 (이메일, 프사, 성별, 생일 등)을 동의받도록 이곳에서 설정하면 된다.

### 3-2. 우리 앱에서 kakao SDK 사용하기

카카오 SDK를 사용하기 위한 사전 준비를 마쳤으니, 이제 우리 프로젝트에서 사용할 시간이다.

먼저 카카오 SDK를 사용하는 Activity인 AuthCodeHandlerActivity를 선언해야 한다. 아래 코드를 Manifest에 추가하면 된다.

```xml
<activity 
    android:name="com.kakao.sdk.auth.AuthCodeHandlerActivity"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        
        <!-- Redirect URI: "kakao${NATIVE_APP_KEY}://oauth" -->
        <data android:host="oauth"
                android:scheme="kakao${NATIVE_APP_KEY}" />
    </intent-filter>
</activity>
```

참고로 여기서 AuthCodeHandlerActivity는 개발자가 생성하는 것이 아니라 카카오 SDK에 이미 포함되어있는 클래스이니, 괜히 com.kakao.sdk.auth 경로에 들어가 AuthCodeHandlerActivity를 생성하는 불상사를 일으키지 말자.

그리고 "kakao${NATIVE\_APP\_KEY}" 부분은 local.properties에서 관리되고 있는 네이티브 앱 키를 사용하면 된다.

다음으로 로그인을 관리할 Activity를 선언해 준 후, 아래 코드를 사용해 주자.

```kotlin
// 로그인 조합 예제

// 카카오계정으로 로그인 공통 callback 구성
// 카카오톡으로 로그인 할 수 없어 카카오계정으로 로그인할 경우 사용됨
val callback: (OAuthToken?, Throwable?) -> Unit = { token, error ->
    if (error != null) {
        Log.e(TAG, "카카오계정으로 로그인 실패", error)
    } else if (token != null) {
        Log.i(TAG, "카카오계정으로 로그인 성공 ${token.accessToken}")
    }
}

// 카카오톡이 설치되어 있으면 카카오톡으로 로그인, 아니면 카카오계정으로 로그인
if (UserApiClient.instance.isKakaoTalkLoginAvailable(context)) {
    UserApiClient.instance.loginWithKakaoTalk(context) { token, error ->
        if (error != null) {
            Log.e(TAG, "카카오톡으로 로그인 실패", error)

            // 사용자가 카카오톡 설치 후 디바이스 권한 요청 화면에서 로그인을 취소한 경우,
            // 의도적인 로그인 취소로 보고 카카오계정으로 로그인 시도 없이 로그인 취소로 처리 (예: 뒤로 가기)
            if (error is ClientError && error.reason == ClientErrorCause.Cancelled) {
                return@loginWithKakaoTalk
            }

            // 카카오톡에 연결된 카카오계정이 없는 경우, 카카오계정으로 로그인 시도
            UserApiClient.instance.loginWithKakaoAccount(context, callback = callback)
        } else if (token != null) {
            Log.i(TAG, "카카오톡으로 로그인 성공 ${token.accessToken}")
        }
    }
} else {
    UserApiClient.instance.loginWithKakaoAccount(context, callback = callback)
}
```

_카카오 로그인 공식문서에서 제공한 코드다. 이 코드를 그대로 쓰지는 말고 각각의 분기마다의 역할을 이해하고 함수를 분리해서 사용하자!_

위의 코드를 보면 분기가 상당히 많다. 코드를 하나하나 읽고 이해가 가지 않는 부분은 블로그와 ChatGPT를 참고하며 각각의 분기마다의 동작을 이해했다.

각각의 분기를 이해하기 위해선 우선 카카오 로그인 방식을 이해할 필요가 있다. 카카오에 로그인하는 방식은 카카오톡으로 로그인 하는 방법과 웹페이지를 띄워 카카오 계정의 ID와 PassWord를 입력받아 로그인하는 방법의 두 가지 방식이 있다. 각각의 방식은 유저의 디바이스에 카카오톡이 설치되어 있는지 여부에 따라 달라진다. 다음의 상황에 따라 다른 방식으로 로그인하게 된다.

1.  사용자의 디바이스에 카카오톡이 설치되어 있지 않다면, 카카오 계정으로 로그인을 시도한다.
2.  사용자의 디바이스에 카카오톡이 설치되어 있지만 계정은 연결되어 있지 않다면, 카카오 계정으로 로그인을 시도한다.
3.  사용자의 디바이스에 카카오톡이 설치되어 있고 계정이 연결되어 있다면 카카오톡으로 로그인을 시도한다.

<img src=https://github.com/user-attachments/assets/bd116e40-8623-460c-8229-863e4c0d0de6 width=700 />


카카오 공식문서에서 친절하게 각 상황에 대한 주석을 달아두었으니, 쉽게 이해할 수 있을 것이다.

```kotlin
UserApiClient.instance.loginWithKakaoTalk(context) { token, error ->
    if (error != null) {
        Log.e(TAG, "로그인 실패", error)
    }
    else if (token != null) {
        Log.i(TAG, "로그인 성공 ${token.accessToken}")
    }
}
```

성공적으로 카카오에 로그인했다면, 카카오 API가 OAuthToken 객체를 만들고 이 객체가 Callback 함수의 token 파라미터에 전달된다. OAuthToken 객체의 accessToken 프로퍼티에 카카오 Access Token이 들어있으므로 내가 이것을 총대마켓 서버로 보내주면 클라이언트에서 할 일은 끝난다! 이제부터는 백엔드 개발자들의 차례다. 이제 총대마켓 서버가 클라이언트에서 보내준 카카오 Access Token을 가지로 카카오 서버와 통신해 내 카카오 유저 정보를 내려받으면 되는 것이다! 👍

## 4. 삽질 기록

이 일련의 과정을 단 한 번의 삽질도 없이 끝냈다면 몇 시간 만에 끝났을 것이다..😁 하지만 실제론 삽질을 하느라 밤을 새웠다! 과정이 많고 알아야 할 지식이 많았던 만큼 여기저기 헤맸다.. 그것을 여기 기록해 보겠다!

### 1) AuthCodeHandlerActivity 이 녀석 뭐지??

카카오 SDK를 사용하기 위해 다음 코드를 Manifest에 추가하면서 문제가 생겼다.

```xml
<activity 
    android:name="com.kakao.sdk.auth.AuthCodeHandlerActivity"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        
        <!-- Redirect URI: "kakao${NATIVE_APP_KEY}://oauth" -->
        <data android:host="oauth"
                android:scheme="kakao${NATIVE_APP_KEY}" />
    </intent-filter>
</activity>
```

나는 처음에 이 코드를 추가하면서 당연히 com.kakao.sdk.auth 경로에 AuthCodeHandlerActivity를 내가 만들어야 하는 줄 알았따..

<img src=https://github.com/user-attachments/assets/77648ecf-1281-4eac-bb91-7451f56cde84 width=300 />


그래서 이렇게 패키지까지 하나씩 추가해 주고 직접 만든 AuthCodeHandlerActivity를 넣어주었다..

<img src=https://github.com/user-attachments/assets/88fd7054-e49b-49aa-9e8a-93d17ec2670c width=400 />


Activity를 만들었으니 뭔가를 넣어야겠지?라고 생각했지만 공식문서에 해당 내용은 찾을 수 없었다.. 그래서 다른 블로그를 참고했는데, 이 블로그에서도 잘못된 정보를 공유한 바람에 혼란이 가중되었다.. 😭 (아마 나와 같은 생각으로 그분도 AuthCodeHandlerActivity를 직접 만들어야 한다고 생각한 모양이다..!)

이렇게 했을 때 로그인이 안 되길래 여러 블로그를 돌아다니면서 한참을 헤매다가.. 나중에 같은 우테코 안드로이드 크루인 케이엠에게 물어봤더니 이 AuthCodeHandlerActivity 가 내가 직접 만들어주는 것이 아니라는 것을 알게 되었고.. 문제를 해결할 수 있었다.

### 2) 카카오 로그인을 하기 위해 CI가 필요한데.. CI를 받아올 수가 없네!?

나뿐만 아니라 백엔드 팀원들과도 같이 고민했던 부분이다.

클라이언트가 카카오 로그인을 한 후 서버로 카카오 Access Token을 넘겨주게 되는데, 처음에 구현하려고 했던 방법은 이것이 아니었다. 처음에 서버에서는 카카오 CI(Connection Information)을 요구했다. 이 당시에는 나와 백엔드 팀원 모두 소셜 로그인 플로우에 대해 잘 몰랐기 때문에 꼭 CI가 필요하다고 생각했다. 유저마다 고유한 카카오 CI가 존재하고, 이를 서버에 보내면 서버가 해당 CI의 총대마켓 계정을 생성하는 방식으로 구현할 수가 있었다. 그런데..

<img src=https://github.com/user-attachments/assets/2013934e-ebab-4578-ab8d-c7c91f528862 width=800 />


어..? CI(연계정보)가 권한이 없다??

더 알아보니 CI는 사업자 등록을 해야만 받아올 수 있는 권한이 생기는 정보였다.. 우리가 필요한 데이터는 보안성을 위해 타인이 쉽게 유추할 수 없어야 하며, 중복이 없어야 하므로 유저마다 고유한 정보여야 했다.

하지만 CI를 제외하면 이 조건을 만족하는 유저 데이터가 없었다. 그나마 유저마다 고유한 값인 이메일을 임시로 사용하기로 했지만.. 유추하기가 너무 쉬웠다.

그래서 이 문제에 대해 논의하던 중.. 우연히 점심을 같이 먹게 된 제이슨 코치님과 크루 케이엠으로부터 카카오 Access Token으로 서버가 카카오와 통신하는 방법을 쓰는 게 좋겠다는 의견을 받았다. 클라이언트가 아닌 서버가 카카오와 통신하는 방식으로 변경할 경우 다음과 같은 장점들을 얻을 수 있었다.

1.  카카오 Access Token은 매 로그인 시마다 바뀌므로 보안 상 더욱 안전하다.
2.  필수로 동의받아야 하는 사용자 개인정보가 없으므로 사용자가 더욱 안심하고 우리 서비스를 사용할 수 있다. (소소한 장점..!)

그렇게 우리는 카카오 Access Token만 사용하는 방식으로 변경했고, 성공적으로 안전한 로그인을 구현할 수 있었다~~


## 참고
https://developers.kakao.com/docs/latest/ko/kakaologin/android

https://developers.kakao.com/

