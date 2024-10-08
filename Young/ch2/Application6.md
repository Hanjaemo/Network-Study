## 비디오 스트리밍과 콘텐츠 분배 네트워크
현재 인터넷 프로토콜의 80%는 비디오 스트리밍이 차지하고 있다. 이런 비디오 스트리밍 서비스가 어떻게 구현되는지 알아보자

### 1. 인터넷 비디오
녹화된 비디오(영화, 드라마 등)은 서버에 저장되어 사용자가 **온디맨드로 요청**한다.
여기서 비디오 매체란 초당 24~30개의 이미지를 보여주는 이미지 연속이다. 
- 압축되지 않은 디지털 인코딩된 이미지는 픽셀 단위이다.
- 각 픽셀은 휘도와 색상을 나타내는 여러 비트들이다.

비디오의 특징은 **압축 가능하다는 것인데** _압축될수록 비디오의 품질과 비트전송률은 떨어지고_, _압축이 덜할수록 비디오의 품질과 비트 전송률이 높아진다_. 중요한 점은 **비트 전송률이 높다면 비디오 품질도 높아진다는 것이다.** 즉, high-end 동영상의 경우, 비트 전송률이 높아지고 트래픽과 스토리지 용량이 엄청나게 필요하다는 것을 의미한다.
스트리밍 비디오에서 가장 중요한 점은 **연속된 비트를 전달하기 위해서는 압축된 비디오의 전송률 이상의 처리량을 제공해야 한다**는 것이다.


### 2. HTTP 스트리밍 및 DASH
HTTP 스트리밍에서 비디오는 HTTP 서버 내에서 특정 URL을 가지는 일반적인 파일로 저장된다. 클라이언트에서는 TCP 연결 이후 HTTP GET을 통해 해당 URL을 요청하고 서버는 비디오 파일을 전송한다. 그리고 클라이언트에서는 **애플리케이션 버퍼**에 전송된 바이트가 저장된다. 

**애플리케이션 버퍼**
- 이 버퍼의 임계값(threshold)를 초과하면 클라이언트 애플리케이션 재생을 시작한다.
- 애플리케이션이 버퍼에서 주기적으로 비디오 프레임을 가져와 압축해제하여 화면에 보여준다.

HTTP 스트리밍의 단점은 모두 같은 비디오를 전송받는다는 것이다. 클라이언트 별로 **가용 대역폭**이 다르고, 시간에 따라 바뀌는데 같은 비디오를 전송받는 다면 속도 차이가 발생한다. 따라서 HTTP 기반 스트리밍인 **DASH**가 개발되었다.

**DASH**
- 비디오가 여러 버전으로 인코딩되며 각 버전은 비트율과 품질 수준이 서로 다르다.
- 클라이언트는 비디오 조각(chuck) 단위로 요청하여 가용 대역폭에 따라 매번 다른 비디오 버전이 선택된다.
- DASH를 사용할 떄 각 비디오 버전은 HTTP 서버에 각각 다른 URL을 가지고 저장된다. HTTP 서버는 비트율에 따라 각 버전의 URL을 제공하는 **매니페스트 파일(manifest file)을 가지고 있다**.

1. 클라이언트는 매니페스트 파일을 요청하여 다양한 버전에 대해 알게 된다.
2. 원하는 버전의 비디오 조각 단위 데이터를 선택하여 HTTP GET 요청에 URL과 byte-range를 지정하여 요청한다.
3. 다운로드하는 동안에 수신대역폭과 비트율 결정 알고리즘을 이용해 다음 버전을 선택한다.
-> DASH는 클라이언트가 서로 다른 품질 수준을 자유롭게 변화시킬 수 있도록 해준다.

### 3. 콘텐츠 분배 네트워크(CDN)
<span style="background-color:#FFF6B9">**하나의 거대한 데이터 센터 구축 및 직접 전송하기**</span>
**_문제점_**

1. **클라이언트와 데이터 센터가 물리적으로 멀 경우**, 여러 통신 링크와 ISP를 거치는데 이떄 하나라도 비디오 소비율보다 낮은 전송용량을 가직게 될 경우 **처리율이 낮아져서 비디오가 멈추는 현상**이 발생한다.
2. 인기 있는 비디오는 같은 통신 링크를 통해 여러 번 반복적으로 전송된다. -> 중복 비용 발생
3. **데이터 센터가 하나만 있는 경우 한 번의 장애로 전체 서비스가 중단될 수 있다는 위험이 존재**한다.

이를 해결하기 위해 대부분의 비디오 스트리밍 애플리케이션은 CDN을 사용한다.

#### CDN
다수의 지점에 분산된 서버를 운영하며, 복사본을 이러한 분산 서버에 저장한다. CDN은 사설 CDN일수도 있고, 제 3자가 운영하는 third-party CND일수도 있다.

- Enter Deep : 서버 클러스트를 접속 네트워크에 구축함으로써 ISP 접속 네트워크 깊숙히 들어가는 것이다. **서버를 최대한 사용자에게 가까이 위치시키지만, 고도로 분산되어 있기 때문에 서버 클러스터를 유지하는 비용이 커진다.**
- Bring Home : 좀 더 적은 수의 핵심 지점에 큰 규모의 서버 클러스터를 구축하여 ISP를 Home으로 가져오는 개념이다. 접속 ISP에 연결하는 대신 CDN 클러서트를 인터넷 교환지점에 배치한다. **클러스터 유지 및 관리 비용일 줄어들지만, 사용자에게서 멀어진다.**

CDN은 push가 아닌 pull 방식을 이용하여 사용자가 지역 클러스터에 없는 비디오를 요청하면 다른 클러스터나 중앙 서버로부터 전송받아 전달하며 이를 자신의 서버 클러스터에 저장한다. 


#### CDN의 동작
1. 웹 브라우저가 특정 비디오의 재생을 요청
2. CDN이 이를 가로채서 적당한 CDN 클러스터에 연결됨
3. 클라이언트의 요청을 해당 클러스터 서버로 전달
---
![](https://velog.velcdn.com/images/choiyoung6609/post/bb3ab6d7-db4b-4eb4-847e-b9ce5e45154c/image.png)

1. 사용자가 A 웹페이지를 방문한다.
2. 비디오 링크를 클릭하면 해당 url에 대한 DNS 질의를 보낸다.
3. LDNS는 url에서 video문자열ㅇ르 감자하고 해당 질의를 A의 책임 DNS 서버로 전달한다. 해당 DNS 질의를 A CDN으로 연결하기 위해 IP 주소 대신에 A CDN의 호스트 이름을 알려준다.
4. DNS 질의는 A CDN의 사설 DNS 구조로 들어가게 된다. LDNS는 두 번째 질의를 보내고 A CDN의 DNS에 의해 A CDN의 콘텐츠 서버의 IP 주소로 변환되어 LDNS에 응답된다.
5. LDNS는 CDN 서버의 IP 주소를 호스트에게 알려준다.
6. 클라이언트는 A CDN 서버의 IP 주소를 얻고 나면, 해당 주소로 TCP 연결을 신청하고 HTTP GET 요청을 보낸다. 만약 DASH가 사용된다면 서버는 다른 각기 다른 버전의 URL 목록을 포함하는 매니페시트 파일을 클라이언트에게 전송하고 동적으로 다른 버전이 선택된다.'

#### 클러스터 선택 정책
CDN은 클라이언트가 DNS 서비스를 수행하는 과정에서 클라이언트의 LDNS 서버의 IP 주소를 얻게 된다. 클러스터를 선택하는 방법 중에는 해당 IP 주소에 기초해 클러스터를 선택하는 것이다.

> 지리적으로 가장 가까운 클러스터 할당
1. Quova 같은 상용 지리정보 데이터베이스를 사용하면 LDNS의 IP 주소는 지리적으로 매핑되어 LDnS와 가장 가까운 CDN이 선택된다.
2. 그러나, 네트워크 경로의 길이 홉에 따라 가장 가깝지 않을 수도 있고, 일부 사용자는 LDNS가 상당히 멀 수도 있기 때문에 잘 동작하지 않을 수 있다.
3. 또한 인터넷 경로 지연과 가용 대역폭을 무시한 채 항상 같은 클러스터를 할당한다.

-> 현재에는 클라이언트와 클러스터 간의 지연 및 손실 성능에 대한 **실시간 측정**이 이루어져 최선의 클러스터를 선택하도록 한다.


### 사례연구 : 넷플릭스, 유튜브
#### 넷플릭스
넷플릭스에는 여러 로그인, 결제, 검색 등 다양한 기능을 처리하는 웹사이트가 존재한다. 이는 아마존 서버 안에 있는 아마존 클라우드에서 실행된다.
또한 다음과 같은 기능도 한다.
- **콘텐트 수집** : 비디오 분배 전 영화의 스튜디오 마스터 버전을 받아 호스트에 업로드
- **콘텐츠 처리** : 여러 가지 형식의 비디오 생성. 각 형식별로 다양한 비트율의 버전 생성
- **CDN으로의 버전 업로드** : 아마존 클라우드는 여러 버전의 비디오를 CDN에 업로드

넷플리스가 자체 CDN을 구축하기 위해 IXP 및 거주용 ISP 자체에 서버 랙을 설치했다.
IXP 설치에는 수십 개의 서버와 DASH를 지원하는 여러 버전의 비디오를 포함한 전체 스트리밍 비디오 라이브러리가 포함된다. 넷플릭스는 풀 캐싱을 사용항 IXP, ISP, CDN 서버를 채우고 사용량일 적을 때 비디오를 CDN 서버에 푸시하여 배포한다.

넷플릭스는 **CDN 서버를 알기 위해서 DNS 리다이렉셔능ㄹ 사용하지 않고**, 아마존 클라우드에서 실행 중인 넷**플릭스 소프트웨어가 CDN 서버를 결정**한다. 최적의 서버를 결정하기 위해서, 클라이언트가 ISP에 설치된 CDN 서버 랙에 있는 **로컬 ISP**를 사용하고, 이 랙에 요청된 사본이 있는 경우 일반적으로 이 랙 서버가 선택되고 **그렇지 않는다면 IXP 서버가 선택**된다.

#### 유튜브
넷플릭스와 비슷하게 구글은 자체 비공개 CDN을 사용하여 유튜브 동영상을 배포하고, 수백 가지 IXP 및 ISP 위치에 서버 클러스터를 설치했다. 대신 구글은 사용자를 특정 서버 클러스터와 연결하는데 DNS를 사용한다. **연결 정책은 클라이언트와 클러스터 간의 RTT가 가장 적은 곳으로 연결하는 것**이다.

유튜브는 여러 품질의 비디오 버전을 제공하지만 DASH 대신에 **사용자가 스스로 버전을 선택하도록 만들었다**. -> **재생 위치 조정과 조기 종료로 인한 대역폭과 서버 자원의 낭비**를 줄이기 위해 유튜브는 HTTP byte-range 헤더를 이용해 목표한 분량의 선인출 데이터 이후에 추가로 전송되는 데이터 흐름을 제한한다.
