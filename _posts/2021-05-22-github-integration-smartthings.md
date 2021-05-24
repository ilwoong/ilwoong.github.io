---
layout: post
title: SmartThings GitHub Repo에서 DTH 가져오기
tags: [SmartThings, DTH, 다원 가스락]
author: ilwoong
---

국내에서 시판중인 상당수의 IoT 장비들은 이미 SmartThings에서 기본적으로 지원하고 있지만, 아직 지원하지 않는 제품들도 있습니다. 이러한 장비들을 SmartThings에서 사용하려면 DTH(Device Type Handler)를 별도로 지정해줘야 합니다. DTH는 PC에서 장치 드라이버와 유사한 기능을 한다고 생각하면 됩니다.

### GitHub에서 DTH 가져오기

0. [GitHub](https://github.com) 계정이 없다면 계정을 생성합니다.

1. [SmartThings IDE](https://graph.api.smartthings.com/) 가입하고 로그인합니다.

2. My Locations 메뉴에서 DTH를 설치할 장소 선택합니다. 장소가 하나밖에 없어도 클릭해서 선택해야 합니다.(ex. 집)

3. My SmartApps 메뉴에서 Enable GitHub Integration 선택합니다.

4. GitHub 로그인창이 나오면 로그인하여 계정을 연동합니다.

5. My Device Handlers 메뉴에서 Update from Repo를 선택하고, 설치하고자 하는 DTH를 선택한 다음 Publish에 체크하고 완료합니다.


### 예제: 다원 IoT 가스락

2021년 5월 22일 현재 다원 IoT 가스락은 아직 SmartThings에서 자동으로 인식하지 않고 있습니다. SmartThings 허브에 연결할 때 Thing으로 등록한 다음 DTH를 변경할 수도 있지만, 먼저 DTH를 등록해놓고 연결하면 자동으로 인식하게 할 수 있습니다.

SmartThings에서 사용하기 위하여 위 단계의 5번에서 다음 DTH 를 검색하여 체크합니다. 

```
devicetypes/smartthings/zigbee-valve.src/zigbee-valve.groovy
```

29번째 행 다음에 아래와 같이 추가합니다.

```
fingerprint profileId: "0104", inClusters: "0000, 0001, 0003, 0004, 0005, 0006", outClusters: "0003, 0019", manufacturer: "DAWON_DNS", model: "SG-V100-ZB", deviceJoinName: "Dawon Gas Valve" //Smart Gas Valve Actuator
```

Save 버튼을 눌러 저장하고, Publish > For Me 버튼을 눌러서 발행하면 다원 가스락의 DTH 설치가 완료됩니다.  이제 가스락을 SmartThings 허브에 연결하면 됩니다.

### 참고자료
- <https://yourjune.tistory.com/1043>
- <https://blog.naver.com/fromzip/222265871770>