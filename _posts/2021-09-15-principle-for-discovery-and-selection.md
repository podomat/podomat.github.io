---
title: "NF 및 NF service의 discovery와 selection"
date: 2021-09-15
categories:
  - Tech
tags:
  - 5G
  - SCP
---

# General

NF 또는 SCP는 연동할 특정 타입 NF의 Service Instance를 찾기 위해 NRF가 제공하는 `NF/NF Service Discovery`를 이용합니다.

**NOTE:** NRF는 SCP와 함께 위치할 수 있습니다.
{: .notice}

NRF가 discovery 서비스를 제공하기 위해서는 `NF Profile`이 NRF에 등록되어 있어야 합니다.
`NF Profile`은 NRF가 (NF/NF Service Instance가 처음 동작할 때 같은 경우에) `NF Profile`이 포함된 NFRegister 메시지를 받아서 등록합니다.

NF 또는 SCP가 등록된 `NF/NF Service`에 대한 정보를 얻기 위해 원하는 `NF type` 정보를 전달하고, 추가로 특정 서비스에 대한 목록이나 slice 정보 등을 제공할 수 있습니다.
또한, `NF Set` 관련 정보를 제공하여 `NF Set` 내에서 NF Instance를 re-selection 할 수 있게 합니다.

가입자 데이터에 접근하는 NF(ex, HSS, UDM)의 경우, NRF는 제공된 가입자ID에 해당하는 `NF Group ID`를 확인할 수 있어야 합니다.
NRF가 이 정보(가입자ID에 대한 NF그룹ID의 매핑)를 가지고 있지 않다면,
UDR이 제공하는 `Nudr_GroupIDmap_Query` 서비스를 이용하여 가입자ID에 해당하는 `NF Group ID`를 조회할 수 있습니다.

Indirect Communication 방식에서 NF는 Request 메시지를 타겟에 전달하기 위해 SCP를 이용합니다.
Delegate Discovery 모드로 설정된 NF는 discovery 절차를 생략하고 SCP에 이 절차를 위임합니다.
이때, NF는 SCP가 discovery 및 selection 하는데 필요한 파라미터를 Request 메시지에 포함합니다.
SCP는 NRF를 연동하여 discovery 결과를 얻고, NRF나 UDR을 연동하여 가입자ID에 해당하는 `NF Group ID`를 얻습니다.

**NOTE**: SCP는 HSS/UDM에 대한 delegated discovery 시에 NRF에게 사용자 ID를 전달하고 HSS/UDM 인스턴스 그룹 정보를 찾거나
아니면 UDR에 HSS/UDM Group ID를 질의할 수 있습니다.
{: .notice}

NRF는 discovery 조건에 해당하는 NF/NF Service Instance 목록을 제공하고,
NF Instance의 IP주소나 FQDN, NF Service Instance와 관련된 endpoint 주소들을 NF나 SCP에게 제공합니다.
그리고, `NF Set ID`나 `NF Service Set ID`도 제공합니다.
discovery 응답에는 제공된 discovery 결과가 캐쉬될 수 있도록 유효기간을 포함합니다.
유효기간 동안에는 조건이 같다면 discovery 결과도 어떤 가입자에게나 동일하므로,
discovery 응답으로 수신한 `NF Profile`들의 정보는 캐쉬되었다가 NF selection에 사용될 수 있습니다.

Direct Communication 방식에서 NF는 discovery 결과를 이용하여 NF/NF Service Instance를 선택합니다.

Delegated Discovery가 없는 Indirect Communication 방식에서 NF는 discovery 결과를 사용하여 NF Instance를 선택하는 반면,
연관된 NF Service Instance의 선택은 NF 또는 SCP에 의해 수행될 수 있습니다.

위의 두 경우 모두에서 NF는 subsequent selection을 위해 캐쉬된 discovery 결과를 사용할 수 있습니다
즉, NF가 selection을 위해 discovery 절차를 다시 수행할 필요가 없습니다.

Delegated Discovery를 사용한 Indirect Communication 방식에서
SCP는 NF가 제공한 discovery 및 selection 파라미터를 기반으로 필요하다면 NRF와 연동하여 적절한 NF/NF Service Instance를 검색하고 선택합니다.
사용될 NRF는 NF Consumer가 discovery 파라미터의 일부로 제공할 수 있습니다. (NSSF 쿼리의 결과 등으로 얻어짐)
SCP는 subsequent selection을 위해 캐쉬된 discovery 결과를 사용할 수 있습니다.
즉, SCP가 selection을 수행하기 위해 discovery 절차를 다시 수행할 필요가 없습니다.

**NOTE**: 하나의 PLMN에 Direct/Indirect Communication을 각각 또는 모두 적용할 수 있습니다.
{: .notice}

NF 또는 SCP는 NF Profile의 변경(ex, NF Service Instance가 서비스를 시작 또는 중지함) 또는
새로 등록 및 등록 취소 된 NF Instance에 대한 notification을 수신하기 위해 NRF에 구독 할 수 있습니다.

여러 PLMN에 걸친 NF/NF Service Discovery의 경우, 로컬 PLMN의 NRF는 원격 PLMN의 NRF와 연동하여
discovery 조건에 맞는 원격 PLMN의 NF Instance의 NF Profile을 검색합니다.
로컬 PLMN의 NRF는 NF가 제공한 PLMN ID를 사용하여 타겟 PLMN 특정 쿼리를 만들어서 원격 PLMN의 NRF에 전달합니다.

## Binding 과 Selection 그리고 Re-selection

Binding은 NF Service Instance의 Selection, Reselection을 위한 적절한 타겟 NF Producer Instance를 지정하여,
특정 NF Producer의 자원(context)과 NF Service에 연관된 후속 request 메시지를 전달하기 위해 사용될 수 있습니다.
이를 통해 NF Producer는 (내부 정책 등의 기준에 따라) NF Consumer가 특정 Context에 대해서
NF Service Instance, NF Instance, NF Service Set 또는 NF Set에 바인딩 되어야 함을 나타낼 수 있습니다.

Binding은 또한 적절한 Notification 대상 NF Consumer Instance가 reselection 되도록 NF Consumer가 사용할 수 있고,
특정 Notification 구독에 연관된 후속 Notification 요청 메시지를 전달하고,
NF Service Producer가 후속 메시지를 보낼 가능성이 있는 경우,
해당 data context에 대해서 NF Consumer가 Producer가 되는 Service에 대한 Binding Indication을 제공합니다.

Routing Binding Indication은 Subscribe/Notification 요청 메시지에 포함될 수 있습니다.
이것은 SCP가 메시지를 전달하는 Indirect Communication 방식에서 사용될 수 있습니다.
Routing Binding Indication은 Binding Indication의 복사본입니다.

**NOTE**: Subscribe 요청 메시지는 Binding Indication과 Routing Binding Indication을 모두 포함할 수 있습니다.
{: .notice}

NF Service Producer는 NF Service Consumer에게 Direct/Indirect Communication 절차 중에 관련된
후속 서비스 요청 메시지에서 사용할 수 있도록 Binding Indication을 제공할 수 있습니다.
NF Service Producer가 NF Consumer에게 제공하는 Binding Indication Level은 NF Service Producer의 자원(context)이
NF Service Instance, NF Instance, NF Service Set, NF Set에 바인딩 되어 있는지 여부를 나타냅니다.
Binding Indication은 NF Consumer나 SCP가 NF Service Producer를 (re-)selection 하는데 사용하도록
NF Service Set ID, NF Set ID, NF Instance ID, NF Service Instance ID를 포함할 수 있습니다.
만약, NF Service Producer에서 자원이 생성되었으면, NF Service Producer는 NF Service Producer의 endpoint 주소를 포함시킨 자원 정보를 제공합니다.
Indirect Communication 인 경우, NF Service Consumer가 Delegated Discovery가 제외된 Indirect Communication에서 reselection을 수행하지 않으면
NF Service Consumer는 Binding Indication을 Request나 Subscribe 메시지의 Routing Binding Indication에 복사합니다.

묵시적/명시적 Notification 구독 중에, NF Service Consumer는 NF Service Producer에게 Binding Indication 및 Notification Endpoint를 제공할 수 있습니다.
NF Service Consumer는 Notification 응답에도 Binding Indication을 제공할 수 있습니다.
NF Service Consumer가 NF Service Producer에게 제공하는 Binding Indication Level은 Notification Endpoint가 NF Service Instance, NF Instance, NF Service Set, NF Set 중 어디에 바인딩 되어 있는지를 나타냅니다.
Binding Indication은 NF Set ID, NF Instance ID, NF Service Set ID, NF Service Instance ID 중에 적어도 하나를 포함해야 하고 Service Name도 포함할 수 있습니다. 
NF Service Set ID, NF Service Instance ID, Service Name은 NF Service Consumer에서 Notification을 처리할 서비스와 관련이 있습니다.

**NOTE**: NF Service는 이 규격에 따라 표준화된 서비스이거나 커스텀 서비스일 수 있습니다.
커스텀 서비스는 NRF에서 Notification을 수신할 Endpoint 주소를 등록하기 위한 용도로만 사용할 수 있습니다.
{: .notice}

NF Service Producer가 Notification을 전송할 때 Binding Indication을 사용하여 Endpoint 주소를 Reselect하고 Notification Endpoint(Notification이 전송될 URI)를 만듭니다.
예를 들어, Subscription에 포함되어 제공된 NF Service Consumer의 Notification Endpoint에 접속이 되지 않으면 다음과 같이 동작합니다.

- Binding Indication에서 Service Name이 생략되고 Notification의 Binding이 NF Set이나 NF Instance 레벨이면,
Binding Indication에 의해 선택된 NF들에 NF Profile 레벨의 NRF에 등록된 endpoint 주소를 사용하여 새로운 Notification Endpoint를 만들어야 합니다.
- Binding Indication에 Service Name이 포함되어 있으면, Binding Indication에 의해 선택된 NF Profile들에
해당 서비스의 NRF에 등록된 endpoint 주소를 사용하여 새로운 Notification Endpoint를 만들어야 합니다.

Indirect Communication에서 NF Service Producer는 SCP가 대체 endpoint 주소를 검색하여 Notification Endpoint를 만들 수 있도록 하기 위해
Binding Indication을 Notification 요청 메시지에 포함되는 Routing Binding Indication에 복사합니다.
예를 들어, Request target인 Notification Endpoint에 접속이 되지 않으면 다음과 같이 동작합니다.

- Routing Binding Indication에서 Service Name이 생략되고 notification의 binding이 NF Set이나 NF Instance 레벨이면, 
Binding Indication에 의해 선택된 NF들에 NF Profile 레벨의 NRF에 등록된 endpoint 주소를 사용하여 새로운 Notification Endpoint를 만들어야 합니다.
- Routing Binding Indication에 Service Name이 포함되어 있으면, Binding Indication에 의해 선택된 NF Profile들에
해당 서비스의 NRF에 등록된 endpoint 주소를 사용하여 새로운 Notification Endpoint를 만들어야 합니다.

다른 Network Function을 통한 Notification Subscription의 경우,
Subscription 관련 이벤트에 대한 별도의 Binding Indication이 NF Service Consumer에 의해 제공될 수 있으며
제공되는 경우 이벤트 관련 구독에 대한 알림을 나타내도록 적용할 수 있어야 합니다.
NF Consumer 로서의 NF가 서비스 요청 메시지에 Produce하는 서비스에 대한 Binding Indicator를 제공하면, 
Binding Indication은 다른 서비스들을 가리키는 것으로 적용되어야 하고 관련된 서비스 이름들을 포함합니다.
그 밖에 나머지 파라미터들은 아래 표에 리스트되어 있습니다.
만약, 서비스 이름이 제공되지 않으면, Binding Indication은 해당 NF가 제공하는 모든 서비스에 해당됩니다.

바인딩의 NF Set 또는 NF Instance Level의 경우, Notification 및 기타 서비스에 대한 Binding Indication은 이것들이 동일한 서비스에 대한 것이라면 결합 될 수 있으며,
결합된 Binding Indication은 Binding Indication과 관련된 모든 시나리오를 나타내는 적용성과 연관되어야 합니다. (이를 위해 적용성은 값의 조합으로 나타낼 수 있습니다.)
요청 또는 구독 메시지에 적용성이 표시되지 않은 경우,
해당 메시지의 Binding Indication은 구독 관련 이벤트를 제외한 모든 이벤트의 Notification에 적용됩니다.

**NOTE**: 이러한 요청 메시지는 묵시적 구독으로 사용할 수 있습니다.<br>
**NOTE**: 요청 메시지는 서비스 및 Notification에 대한 Binding Indication과 (Indirect Communication의 경우) Routing Binding Indication을 모두 포함할 수 있습니다.
{: .notice}

다음 표는 NF Service Producer가 제공하는 Binding Indicator에 따른 NF Service Consumer 및 SCP의 selection/reselection 동작을 정의합니다.

|Level of Binding Indication|The NF Consumer / Notification sender / SCP selects|The NF Consumer / Notification sender / SCP can reselect e.g. when selected producer is not available|Binding information for selection and re-selection|
|---|---|---|---|
|**NF Service Instance**|해당 NF Service Instance|(가능하면) 해당 NF Service Set 내의 동등한 NF Service Instance<br>해당 NF Instance 내의 동등한 NF Service Instance<br>(가능하면) 해당 NF Set 내의 동등한 NF Service Instance|NF Service Instance ID, NF Service Set ID, NF Instance ID, NF Set ID, Service Name (NOTE 4)|
|**NF Service Set**|해당 NF Service Set 내의 임의의 NF Service Instance|(가능하면) 해당 NF Set 내의 동등한 NF Service Set 내의 임의의 NF Service Instance (Note 2)|NF Service Set ID, NF Instance ID, NF Set ID, Service Name (NOTE 4)|
|**NF Instance**|해당 NF Instance 내의 임의의 동등한 NF Service|(가능하면) 해당 NF Set 내의 다른 NF Instance 내의 임의의 동등한 NF Service Instance|NF Instance ID, NF Set ID, Service Name (NOTE 4)|
|**NF Set**|해당 NF Set 내의 임의의 동등한 NF Service Instance|해당 NF Set 내의 임의의 동등한 NF Service Instance|NF Set ID, Service Name (NOTE 4)|
|NOTE 1: Binding Indication을 사용할 수없는 경우, NF Consumer는 사용 가능한 라우팅 정보를 기반으로 서비스 Request message를 대상으로 전송한다.<br>NOTE 2: 다른 NF들의 NF Service Set은 NF Service의 타입 및 Variant(예: 동일한 NF Service Set ID)가 동일한 경우 같은 것으로 간주된다.<br>NOTE 3: Routing Binding Indication을 사용할 수 없는 경우, SCP는 사용 가능한 라우팅 정보를 기반으로 서비스 Request를 대상으로 전송한다.<br>NOTE 4: Binding Indication이 Notification 대상과 관련이 있거나 NF Consumer로서의 NF가 NF가 생성하는 서비스에 대한 Binding Indication을 제공하는 경우에만 서비스 이름이 적용된다.||
[Binding, selection and reselection]

## Indirect communication의 Discovery와 Selection

Indirect Communication을 위해, SCP는 NF와 NF서비스 discovery 및 selection 과 관련하여 다음 기능을 수행합니다.

- Request message에 Routing Binding Indication이 포함되어 있으면,
SCP는 해당 서비스 요청메시지를 위의 표에 기술된 대로 요청된 대상에 전송해야 합니다.
Routing Binding Indication이 없으면 SCP는 NRF 또는 자체 설정으로 해당 NF Set ID를 얻는다.
- Request message 수신자가 이전에 Binding Indication을 제공했으면,
Request message 전송자는 이것을 후속 Request message에 Routing Binding Indication으로 포함해야 한다.

<br>

# SCP의 Discovery와 Selection

- NF는 자신의 serving SCP(s)와 함께 구성됩니다.
- SCP가 배치된 deployment에서, 메시지가 목적지까지 가는 동안 여러 SCP instance를 거치게 됩니다.
SCP가 메시지를 전달할 next hop을 결정(discover and select) 할 때,
NRF가 제공하는 Nnrf_NFDiscovery 서비스를 조회하거나 또는 next SCP에 대한 설정 정보를 이용합니다.
- SCP는 Nnrf_NFDiscovery 서비스를 조회할 때 SCP profile 파라미터를 이용하며, 이때 사용할 파라미터는 네트웍의 구성에 따라 다릅니다.
NRF는 조회 요청된 discovery 파라미터 별로 SCP profile 목록을 리턴합니다.
SCP가 Routing Binding Indication이 포함된 서비스나 notification에 대한 요청 메시지를 수신하고 이것을 다시 next hop SCP에 전달하고자 하면,
전달하는 요청 메시지에 Routing Binding Indication을 반드시 포함시켜야 합니다.

**NOTE**: SCP가 NRF로 부터 수신한 정보를 이용하여 producer에 대한 최적 경로를 찾는 방법은 SCP의 구현, 배포별 구성 및 운영자 정책에 따라 달라집니다.
{: .notice}

- SCP 구성에 따라 SCP가 next hop SCP에게 서비스 요청 메시지를 전달하기로 결정했다면,
다음 SCP에게 NF 또는 서비스 인스턴스의 selection도 위임하고 이를 위한 discovery 및 selection 파라미터를 제공해야 합니다.
