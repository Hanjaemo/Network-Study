## 혼잡 제어의 원리
실제 네트워크가 혼잡해짐에 따라 라우터 버퍼들의 오버플로로 발생하는 손실이 있다고 앞서 애기했었다. 그리고 패킷 재전송이 이러한 손실 증상은 다루지만, **너무 많은 출발지에서 높은 속도로 데이터를 보내려고 시도하는 네트워크 혼잡 원인**을 다루지는 않는다. 네트워크 혼잡 원인을 처리하기 위해서는 **송신자들을 억제하는 매커니즘이 필요**하다.

### 1. 혼잡의 원인과 비용
혼잡 제어가 발생하는 점차 복잡해지는 세 가지 시나리오를 살펴봄으로써 혼잡 제어가 왜 발생하는지 혼잡이 발생하면 어떤 일이 일어나는지에 초점을 맞추자.

> **시나리오 1. 2개의 송신자와 무한 버퍼를 갖는 하나의 라우터**

![](https://velog.velcdn.com/images/choiyoung6609/post/f52b9cdf-0717-439b-bc7f-dc0b5d29aa5e/image.png)

송신자 호스트 A, B가 데이터를 보내는데 이때 두 호스트가 단일 라우터를 공유한다고 하자. (이 단일 라우터의 버퍼는 무제한이라고 가정)

- 호스트 A의 애플리케이션은  λin 바이트/초의 평균 전송률로 연결(이 속도로 데이터 보내고 있음)
- 호스트 B도 또한 같은 속도 λin 바이트/초 전송률로 데이터 전송
- 오류 복구, 흐름 제어 또는 혼잡 제어 수행 안함
- 하위 계층에서의 부가적인 오버헤드 무시 => 라우터에게 제공하는 속도도 λin 바이트/초임
- **호스트 A와 호스트 B가 전송하는 패킷은 라우터와 용량 R의 공유 출력 링크를 통과**
- **라우터는 패킷 도착률이 출력 링크의 용량을 초과하여 입력되는 패킷들을 저장하는 버퍼를 가지고 있음**(무제한)

![](https://velog.velcdn.com/images/choiyoung6609/post/19ecf3c6-6313-44af-8a34-004d22550115/image.png)

- 왼쪽은 수신자 측에서의 **연결 당 처리량**을 나타낸 것이다.
- 0에서 R/2 사이의 전송률은 송신자와 수신자 모두 같다.
- 그러나 라우터와 공유 링크를 호스트 A와 B가 공유하기 때문에 전송률이 R/2 이상이여도 처리량은 단순히 R/2이다.
- 오른쪽 그래프는 링크 용량 근처에서 동작의 결과이다
- 전송률이 R/2에 가까워질수록, 평균 지연은 점점 커진다.
- **전송률이 R/2보다 커질 때, 큐잉된 패킷의 평균 개수는 제한되지 않고 출발지와 목적지 사이의 평균 지연이 무제한된다.**

=> 이상적인 버퍼를 가지고 있어도 패킷 도착률이 링크 용량에 근접함에 따라 큐잉 지연이 커진다.

> **시나리오 2: 2개의 송신자, 유한 버퍼를 가진 하나의 라우터**

![](https://velog.velcdn.com/images/choiyoung6609/post/69d24a33-c3cd-4aeb-86c8-547c8a55b67b/image.png)

- 현실적으로 라우터가 무제한일 수는 없으므로, 버퍼가 유한하다고 가정하고, 각 연결은 신뢰적이라고 가정하자.
- 신뢰적인 연결이기 떄문에 라우터에서 버려지면 송신자에 의해 재전송된다. 
- 원본 데이터 전송률은 λin 바이트/초이지만, 원래 데이터와 재전송 데이터를 포함한 송신률은 λ'in 바이트/초이다.
- **때떄로 λ'in 바이트/초는 네트워크에 제공된 부하**라고 부른다.
![](https://velog.velcdn.com/images/choiyoung6609/post/894a4f93-994d-44eb-ab69-df3a68bf858d/image.png)

- 시나리오 2에서 첫 번째 그림은 바로 호스트에서 라우터의 버퍼 상태를 바악할 수 있다고 가정했을 때이다. 이때는 비현실 적이지만 손실이 발생하지 않아서 처리율은 λin와 같다.

- 두 번째 그림은 손실이 발생해서, 송신자가 재전송하는 시나리오이다. 
- 이런 경우 제공된 부하가 λ'in이 R/2인 경우를 고려할 때 데이터의 전송률이 R/3이라고 가정했다. 
- 이때 알 수 있는 것은 **송신자는 버퍼 오버플로 떄문에 버려진 패킷을 보상하기 위해 재전송을 수행해야 한다는 것**이다.

- 마지막으로, 송신자에서 너무 일찍 타임아웃되어 패킷이 손실되지 않았지만 큐에서 지연되고 있는 패킷을 재전송하는 경우이다. 
- 원래 패킷과 재전송 패킷 모두 수신자에게 전다로디지만, 재전송된 패킷은 버린다. 
- **즉, 커다란 지연으로 인해 타임아웃이 발생하고 송신자의 불필요한 재전송이 발생할 경우, 라우터가 불필요한 복사본을 전송하는데 링크 대역폭을 낭비**하게 된다.
- 제공된 부하가 R/2일 경우, R/4의 점선을 갖는다.

> **시나리오 3: 4개의 송신자와 유한 버퍼를 갖는 라우터, 그리고 멀티홉 경로**

- 마지막 시나리오에서는 4개의 호스트는 2홉 경로를 통해 패킷을 전송한다.
- 모든 호스트는 λin의 값을 가지고 라우터 링크는 R 바이트/초 용량을 가진다.
![](https://velog.velcdn.com/images/choiyoung6609/post/b67c6402-ea93-4cee-b116-a7d711b5f4fd/image.png)
A ~ C 연결은 D~B 연결과 라우터 R1을 공유하고, B~D연결과 라우터 R2를 공유한다. 
- 이때 λin가 매우 작다면 버퍼 오버플로우는 거의 발생하지 않고, 처리량과 제공된 부하량 λ'in은 거의 같다. 
- 약간 λin가 더 커지더라도 버퍼 오버플로우는 거의 발생하지 않고, 처리량과 제공된 부하량이 거의 같다. 
- λin가 매우 큰 경우를 고려해보자. A ~ C 트래픽의 경우 R2에서 λin값과 관계없이 최대 도착률인 R을 가질 수 있다. 
- 만약 λ'in이 매우 크다면 R2의 B~D 트래픽 도착률은 A~C보다 클 수 있다. 
- R2에서는 버퍼 공간을 경쟁해야 하므로 B~D에서 제공된 부하가 크면 클수록 A~D 트래픽이 더 작아진다. **다른 트래픽이 아주 많다면 종단 간 처리율이 0이 될 수도 있다.*
- 제공된 부하를 증가시키면 처리량이 궁극적으로 줄어드는 것은 두 번째 라우터에서 버려질 때마다 첫 번쨰 라우터 작업이 헛된 것으로 돌아가기 때문에 발생한다. 
- 따라서, 첫 번쨰 라우터에서 사용되는 전송용량은 **다른 패킷을 전송하면서 좀 더 유익하게 사용될 수 있다.**
- **패킷을 전송하는데 사용된 상위 라우터에서 사용된 전송 용량은 낭비되기 때문에 혼잡 때문에 또 다른 비용이 발생**한다.

### 2. 혼잡 제어에 대한 접근법
혼잡 제어를 수행하는 두 가지의 광범위한 접근 방식을 살펴보고, 접근 방식을 구현하는 특정 네트워크 구조와 혼잡 제어 프로토콜에 대해 논의해보자

- **종단 간의 혼잡 제어** : 네트워크계층은 혼잡 제어 목적을 위해 트랜스포트 계층에게 어떤 것도 직접적으로 제공하지 않는다. 따라서 트랜스포트 계층에서는 혼잡함을 **단지 관찰된 네트워크 동작에 기초하여 추측**해야 한다. 타임아웃이나 삼중 중복 확인 과 같은 TCP 세그먼트 손실에 따라 **혼잡의 발생 표시로 간주하고 이에 따라 윈도 크기를 줄이며 흐름을 제어한다.** 또한, **증가하는 왕복 지연값을 통해 네트워크 혼잡 증가 지표로 사용한다.**

- **네트워크 지원 혼잡 제어** : 라우터들은 네트워크 안에서 송신자와 수신자에게 모두 직접적인 혼잡 피드백을 제공한다. 즉, ATM ABR 혼잡 제어에서 라우터는 자신의 출력 링크에서 제공할 수 있는 전송률을 송신자에게 명확히 알린다. 

- 인터넷의 기본 버전은 종단 간 접근 방식을 선택했으나 최근에 IP와 TCP가 네트워크 혼잡 제어를 선택적으로 구현할 수 있게 되었다.

![](https://velog.velcdn.com/images/choiyoung6609/post/32372501-7238-4fdf-8156-ac87500611ea/image.png)

네트워크 혼잡 제어에 대한 혼잡 정보는 두 가지 방법 중 하나가 선택되는데 직접 피드백과 간접 피드백이다.
- 직접 피드백은 라우터에서 송신자에게 보내는 것으로 **초크 패킷의 형태**를 갖는다.
- 두 번쨰는 혼잡함을 알리기 위해서 송신자에서 수신자로 흐르는 패킷 안의 특정 필드에 표시하여 나타내는 것이다. **완전한 왕복 시간이 걸린다.**
