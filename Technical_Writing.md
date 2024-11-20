# 네이버 지도 SDK (Android - Kotlin)

## 🛫 서론

이 글은 안드로이드 애플리케이션에서 네이버 지도 SDK를 사용하여 지도를 구현하고, 위치 기반 서비스를 추가하려는 개발자들을 위해 작성되었습니다.
네이버 지도 SDK는 사용자 맞춤형 지도 기능과 다양한 위치 정보 시각화 옵션을 제공하여 앱 개발자들이 보다 쉽게 위치 정보를 활용할 수 있도록 돕습니다.
본 문서에서는 SDK 설정부터 지도 표시, 카메라 이동, 좌표 설정 등의 주요 기능을 단계별로 설명하여, 개발 과정에서 필요한 구체적인 지침을 제공합니다.
이를 통해 개발자는 네이버 지도의 다양한 기능을 이해하고, 효과적으로 구현할 수 있을 것입니다.

## ⚙️️ 설정

네이버 지도 SDK를 사용하기 위해서는 네이버 클라우드 플랫폼에서 클라이언트 ID를 발급받고, 발급받은 ID를 SDK에 지정해야 합니다.

### 클라이언트 ID 발급

1. **네이버 클라우드 플랫폼**에 로그인한 후 **콘솔**에 들어갑니다.

   ![1.png](img%2F1.png)

2. **Products & Services**에서 **AI-Application Service** 하위의 **AI·NAVER API**를 선택합니다.

   ![2.png](img%2F2.png)

3. **Application 등록**을 선택하고 **Maps** 하위의 **Mobile Dynamic Map**을 체크합니다.

   ![3.png](img%2F3.png)

   ![4.png](img%2F4.png)

4. Android 앱 패키지 이름에 네이버 지도 SDK를 사용하고자 하는 앱의 패키지명을 추가하고 등록합니다.

   ![5.png](img%2F5.png)

5. 등록한 애플리케이션의 인증 정보를 선택해 Client ID를 확인합니다.

   ![6.png](img%2F6.png)

### 의존성 추가

네이버 지도 SDK는 https://repository.map.naver.com/archive/maven Maven 저장소에서 배포됩니다.
루트 프로젝트의 build.gradle에 저장소 설정을 추가합니다.

![7.png](img%2F7.png)

![8.png](img%2F8.png)

### 클라이언트 ID 지정

발급받은 클라이언트 ID를 SDK에 지정하면 지도 API를 사용할 수 있습니다.
클라이언트 ID는 두 가지 방식으로 지정할 수 있습니다.

1. AndroidManifest.xml에 지정
   AndroidManifest.xml의 <application> 아래에 <meta-data> 요소를 추가하고, name으로 com.naver.maps.map.CLIENT_ID를, value로 발급받은 클라이언트
   ID를 지정합니다.

   ![9.png](img%2F9.png)

2. API를 호출해 지정
   AndroidManifest.xml을 수정하지 않고 API를 호출해 클라이언트 ID를 지정할 수도 있습니다.
   Application의 onCreate() 내에서 NaverMapSdk.setClient()를 호출해 NaverCloudPlatformClient를 지정합니다.
   NaverMapSdk는 싱글톤 클래스이므로 getInstance()를 사용해 인스턴스를 얻어와야 합니다.

   ![10.png](img%2F10.png)

## 🗺️ 지도

하나의 지도는 뷰 요소와 인터페이스 요소로 구성됩니다. 뷰 요소는 화면에 지도를 나타내는 역할을 하며, MapFragment와 MapView가 여기에 해당합니다.
지도를 다루는 인터페이스 역할을 하는 인터페이스 요소는 NaverMap 클래스가 담당하며, 오버레이를 추가하고 상호작용하는 등 지도와 관련된 기능 대부분을 이 클래스에서 제공합니다.
MapFragment 및 MapView는 개발자가 직접 생성할 수 있으나 NaverMap 객체는 오직 콜백 메서드를 이용해서 얻어올 수 있습니다.
지도 화면은 프래그먼트 및 뷰로 제공됩니다. 이 중 한 가지를 레이아웃에 추가하면 화면에 지도가 나타납니다.

### MapFragment

지도를 화면에 나타내는 방법 중 가장 권장되는 것은 MapFragment를 사용하는 것입니다.

![11.png](img%2F11.png)

MapFragment는 androidx.fragment.app.Fragment의 하위 클래스로, 여타 프래그먼트와 마찬가지로 레이아웃 XML 또는 FragmentTransaction을 사용해 추가할 수 있습니다.

![12.png](img%2F12.png)

![13.png](img%2F13.png)

### MapView

프래그먼트를 사용하지 않고 뷰를 바로 사용할 수도 있습니다.
MapView를 레이아웃에 추가하면 MapFragment를 사용한 것과 마찬가지로 지도가 화면에 나타납니다.

![14.png](img%2F14.png)

MapView를 사용할 때는 반드시 MapView가 포함된 액티비티의 라이프 사이클에 맞추어 onCreate(), onStart(), onResume(), onPause(), onStop(), onDestroy(),
onSaveInstanceState(), onLowMemory()를 호출해야 합니다.
단, MapView가 프래그먼트에 포함될 경우 프래그먼트의 onCreateView() 또는 onViewCreated()에서 onCreate()를, onDestroyView()에서 onDestroy()를 호출해야
합니다.
그렇지 않으면 지도가 정상적으로 동작하지 않습니다.
MapFragment를 사용하면 이러한 절차가 필요하지 않으므로 MapFragment를 사용하는 것을 권장합니다.

![15.png](img%2F15.png)

### NaverMap 객체 얻어오기

MapFragment 및 MapView는 지도에 대한 뷰 역할만을 담당하므로 API를 호출하려면 인터페이스 역할을 하는 NaverMap 객체가 필요합니다.
MapFragment 또는 MapView의 getMapAsync() 메서드로 OnMapReadyCallback을 등록하면 비동기로 NaverMap 객체를 얻을 수 있습니다.
NaverMap 객체가 준비되면 onMapReady() 콜백 메서드가 호출됩니다.

![16.png](img%2F16.png)

## 🧭 좌표

좌표는 카메라나 오버레이의 위치같은 지리적 정보를 표현하기 위한 최소 단위로, 지도를 구성하는 가장 기본적인 요소입니다.
지도와 관련된 많은 API는 좌표 또는 영역을 파라미터로 받거나 반환합니다.
네이버 지도 SDK는 지구 표면의 한 지점을 위도와 경도로 표현하는 지리 좌표계를 기본 좌표계로 사용하며, 이와 관련된 기능을 제공합니다.

### LatLng

LatLng는 하나의 위경도 좌표를 나타내는 클래스입니다.
latitude 속성이 위도를, longitude 속성이 경도를 나타냅니다.
LatLng의 모든 속성은 final이므로 각 속성은 생성자로만 지정할 수 있고, 한 번 생성된 객체의 속성은 변경할 수 없습니다.

![17.png](img%2F17.png)

## 📷 카메라

네이버 지도 SDK는 기기 화면 건너편의 지도를 카메라로 바라보는 방식으로 지도를 표현합니다.
카메라를 이동, 확대 및 축소, 기울임, 회전시킴으로써 화면에 보이는 지도를 자유자재로 움직일 수 있습니다.

### 카메라의 위치

카메라의 위치는 카메라가 놓여 있는 대상 지점의 좌표와 줌 레벨, 기울기, 베어링으로 구성됩니다.

1. 대상 지점
   : target은 카메라가 놓여 있는 대상 지점의 좌표를 나타내는 속성입니다. 대상 지점이 변경되면 지도가 동서남북으로 움직입니다.

2. 줌 레벨
   : zoom은 카메라의 줌 레벨을 나타내는 속성입니다.
   줌 레벨은 지도의 축척을 나타냅니다.
   즉, 줌 레벨이 작을수록 지도가 축소되고 클수록 확대됩니다.
   줌 레벨이 커지면 지도에 나타나는 정보도 더욱 세밀해집니다.

3. 기울임 각도
   : tilt는 카메라의 기울임 각도를 나타내는 속성입니다.
   카메라는 기울임 각도만큼 지면을 비스듬하게 내려다봅니다.
   기울임 각도가 0도이면 카메라가 지면을 수직으로 내려다보며, 각도가 증가하면 카메라의 시선도 점점 수평에 가깝게 기울어집니다.
   따라서 기울임 각도가 클수록 더 먼 곳을 볼 수 있게 됩니다.
   카메라가 기울어지면 화면에 보이는 지도에 원근감이 적용됩니다.
   즉, 화면의 중심을 기준으로 먼 곳은 더 작게 보이고 가까운 곳은 더 크게 보입니다.

4. 베어링 각도
   : bearing은 카메라의 베어링 각도를 나타내는 속성입니다.
   베어링은 카메라가 바라보는 방위를 의미합니다.
   카메라가 정북 방향을 바라볼 때 베어링 각도는 0도이며, 시계 방향으로 값이 증가합니다.
   즉, 동쪽을 바라볼 때 90도, 남쪽을 바라볼 때 180도가 됩니다.

### CameraPosition 객체

CameraPosition은 카메라의 위치를 나타내는 클래스입니다.
CameraPosition의 모든 속성은 final이므로 생성자로만 지정할 수 있고, 한 번 생성된 객체의 속성은 변경할 수 없습니다.

![18.png](img%2F18.png)

CameraPosition 객체를 생성할 때 기울임 각도와 베어링 각도는 생략할 수 있습니다. 생략할 경우 0으로 지정됩니다.

![19.png](img%2F19.png)

### 카메라 이동

카메라는 API 호출, 사용자의 제스처 등 다양한 방법으로 움직일 수 있습니다.
API를 호출해 카메라를 움직이려면 CameraUpdate 객체와 moveCamera() 메서드를 사용합니다.
CameraUpdate의 다양한 속성을 사용해 카메라의 위치, 애니메이션, 콜백 등을 지정할 수 있습니다.

#### API 호출로 카메라 이동하기

카메라를 움직이려면 먼저 카메라를 어떻게 움직일지를 나타내는 CameraUpdate 객체를 생성해야 합니다.
CameraUpdate는 카메라를 이동할 위치, 방법 등을 정의하는 클래스입니다.
CameraUpdate 객체는 오직 팩토리 메서드를 이용해서 생성할 수 있으며, CameraUpdate 클래스에 다양한 팩토리 메서드가 정의되어 있습니다.

- toCameraPosition()
  : 카메라의 위치를 지정한 CameraPosition으로 움직입니다.

- scrollTo()
  : 카메라의 대상 지점을 지정한 좌표로 변경합니다.

- scrollBy()
  : 카메라의 대상 지점을 지정한 픽셀만큼 상하좌우로 이동합니다.

- zoomTo()
  : 카메라의 줌 레벨을 지정한 값으로 변경합니다.

- fitBounds()
  : 영역이 온전히 보이는 좌표와 최대 줌 레벨로 카메라의 위치를 변경합니다.

![20.png](img%2F20.png)

#### 카메라 이동 콜백

finishCallback() 및 cancelCallback()으로 콜백 객체를 지정하면 해당하는 CameraUpdate에 대한 카메라 이동이 완료됐거나 실패했을 때 콜백을 받을 수 있습니다.
카메라 이동에 애니메이션이 지정되어 있고 그 애니메이션이 취소되었을 경우 cancelCallback()에 지정된 콜백이 호출되고, 그 외의 경우에는 finishCallback()에 지정된 콜백이 호출됩니다.
즉, 카메라 이동에 애니메이션이 지정되지 않았으면 카메라 이동은 항상 성공적으로 완료된 것으로 간주됩니다.

![21.png](img%2F21.png)

### 카메라 변경 이벤트

어떤 이유에 의해서건 카메라가 움직이면 카메라 변경 이벤트가 발생합니다.
NaverMap.addOnCameraChangeListener() 메서드로 OnCameraChangeListener를 등록하면 카메라 변경 이벤트를 받을 수 있습니다.
카메라의 위치가 변경되면 onCameraChange() 콜백 메서드가 호출됩니다.

onCameraChange()에는 reason과 animated 파라미터가 전달됩니다.
reason은 이벤트를 발생시킨 카메라 이동의 원인입니다.
CameraUpdate.reason()을 호출해 카메라 이동의 원인을 지정할 수 있으며, 이벤트 리스너 내에서 이 값을 이용해 어떤 원인에 의해 발생한 이벤트인지를 판단할 수 있습니다.
제스처, 컨트롤 등 네이버 지도 SDK의 내장 기능에 의해 카메라가 이동한 경우 미리 정의된 음숫값을 가지며, 개발자가 임의의 양숫값을 지정할 수도 있습니다.

- REASON_DEVELOPER
  : reason의 기본값으로, 개발자가 API를 호출해 카메라가 움직였음을 나타냅니다.

- REASON_GESTURE
  : 사용자의 제스처로 인해 카메라가 움직였음을 나타냅니다.

- REASON_CONTROL
  : 사용자의 버튼 선택으로 인해 카메라가 움직였음을 나타냅니다.

- REASON_LOCATION
  : 위치 트래킹 기능으로 인해 카메라가 움직였음을 나타냅니다.

![22.png](img%2F22.png)

#### 카메라 대기 이벤트

카메라의 움직임이 끝나 대기 상태가 되면 카메라 대기 이벤트가 발생합니다.
NaverMap.addOnCameraIdleListener() 메서드로 OnCameraIdleListener를 등록하면 카메라 대기 이벤트를 받을 수 있습니다.
카메라가 대기 상태가 되면 onCameraIdle() 콜백 메서드가 호출됩니다.

- 카메라가 애니메이션 없이 움직일 때.
  단, 사용자가 제스처로 지도를 움직이는 경우 제스처가 완전히 끝날 때까지(ACTION_UP이 발생할 때까지)는 연속적인 이동으로 간주되어 이벤트가 발생하지 않습니다.
- 카메라 애니메이션이 완료될 때.
  단, 카메라 애니메이션이 진행 중일 때 새로운 애니메이션이 발생하거나, 기존 카메라 이동의 finishCallback() 또는 cancelCallback()으로 지정된 콜백 내에서 카메라 이동이 일어날 경우
  연속적인 이동으로 간주되어 이벤트가 발생하지 않습니다.
- NaverMap.cancelTransitions()가 호출되어 카메라 애니메이션이 명시적으로 취소될 때.

![23.png](img%2F23.png)

## 📍 위치

지도를 사용하는 앱은 사용자의 위치를 추척하고 지도에 표현하는 경우가 많습니다.
네이버 지도 SDK는 이런 기능을 손쉽게 구현할 수 있도록 위치 오버레이와 위치 추적 기능을 제공합니다.
내장된 위치 추적 기능을 사용하지 않고 직접 위치 관련 기능을 구현할 수도 있습니다.

### 권한과 LocationSource

네이버 지도 SDK는 기본적으로 사용자의 위치 정보를 사용하지 않으므로 사용자에게 위치와 관련된 권한을 요구하지 않습니다.
따라서 위치 추적 기능을 사용하고자 하는 앱은 AndroidManifest.xml에 ACCESS_COARSE_LOCATION 또는 ACCESS_FINE_LOCATION 권한을 명시해야 합니다.

![24.png](img%2F24.png)

또한 setLocationSource()를 호출해 LocationSource 구현체를 지정해야 합니다.
LocationSource는 네이버 지도 SDK에 위치를 제공하는 인터페이스입니다.
activate(), deactivate() 등 LocationSource의 메서드는 지도 객체가 호출하므로 개발자가 직접 호출해서는 안됩니다.

### FusedLocationSource

네이버 지도 SDK의 FusedLocationSource는 Google Play 서비스의 FusedLocationProviderClient와 기기의 지자기 및 가속도 센서를 결합하여 최적의 위치 정보를 제공하는 구현체입니다.
이를 통해 GPS 신호가 약하거나 실내 환경에서도 보다 정확한 위치 데이터를 제공받을 수 있습니다.
FusedLocationSource는 다양한 소스로부터 위치 데이터를 수집하고 통합하여 정확도를 높이며, 효율적인 배터리 관리를 돕습니다.
FusedLocationSource를 사용하려면 앱 모듈의 build.gradle에 play-services-location 21.0.1 이상 버전에 대한 의존성을 추가해야 합니다.

![25.png](img%2F25.png)

FusedLocationSource는 런타임 권한 처리를 위해 액티비티 또는 프래그먼트를 필요로 합니다.
생성자에 액티비티나 프래그먼트 객체를 전달하고 권한 요청 코드를 지정해야 합니다.
그리고 onRequestPermissionResult()의 결과를 FusedLocationSource의 onRequestPermissionsResult()에 전달해야 합니다.

![26.png](img%2F26.png)

### 위치 추적 모드

NaverMap에 LocationSource를 지정하면 위치 추적 기능을 사용할 수 있습니다.

- 위치 추적 모드 지정
  : setLocationTrackingMode()를 호출하면 프로그램적으로 위치 추적 모드를 지정할 수 있습니다.

![27.png](img%2F27.png)

- None
  : 위치를 추적하지 않습니다.

- NoFollow
  : 위치 추적이 활성화되고, 현위치 오버레이가 사용자의 위치를 따라 움직입니다.
  그러나 지도는 움직이지 않습니다.

- Follow
  : 위치 추적이 활성화되고, 현위치 오버레이와 카메라의 좌표가 사용자의 위치를 따라 움직입니다.
  API나 제스처를 사용해 임의로 카메라를 움직일 경우 모드가 NoFollow로 바뀝니다.

- Face
  : 위치 추적이 활성화되고, 현위치 오버레이, 카메라의 좌표, 베어링이 사용자의 위치 및 방향을 따라 움직입니다.
  API나 제스처를 사용해 임의로 카메라를 움직일 경우 모드가 NoFollow로 바뀝니다.

- 현위치 버튼 컨트롤 사용
  : UiSettings.setLocationButtonEnabled(true)로 현위치 버튼 컨트롤을 활성화하면 사용자의 클릭에 따라 위치 추적 모드를 변경할 수 있습니다.

### 위치 변경 이벤트

addOnLocationChangeListener() 메서드로 OnLocationChangeListener를 등록하면 위치 변경에 대한 이벤트를 받을 수 있습니다.
위치 추적 모드가 활성화되고 사용자의 위치가 변경되면 onLocationChange() 콜백 메서드가 호출되며, 파라미터로 사용자의 위치가 전달됩니다.

![28.png](img%2F28.png)

## 🏁 결론

네이버 지도 SDK는 개발자가 위치 기반 기능을 간편하게 통합할 수 있도록 다양한 기능과 설정 옵션을 제공하고 있습니다.
강력한 기능과 직관적인 설정 방식은 Android 애플리케이션에 지도를 추가하고 조작하는 과정을 크게 단축해 줍니다.
이를 통해 사용자에게 지도와 관련된 위치 기반 서비스를 효과적으로 제공할 수 있으며, 화면 설정, 좌표 관리, 카메라 조작 등 지도 관련 기능을 유연하게 구성할 수 있습니다.
네이버 지도 SDK를 활용해 앱의 사용자 경험을 더욱 향상시키고, 다양한 위치 기반 기능을 제공할 수 있기를 기대합니다.