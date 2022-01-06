---
title: "NF service 연동을 위한 Communication model"
date: 2021-09-10
categories:
  - Tech
tags:
  - 5G
  - SCP
---

다음은 communication model과 각각의 용법 및 SCP 활용과의 관계를 요약한 것이다.

|Communication between consumer and producer|Service discovery and request routing|Communication model|
|---|---|:---:|
|Direct communication|No NRF or SCP; direct routing|A|
|Direct communication|Discovery using NRF services; no SCP; direct routing|B|
|Indirect communication|Discovery using NRF services; selection for specific instance from the Set can be delegated to SCP. Routing via SCP|C|
|Indirect communication|Discovery and associated selection delegated to an SCP using discovery and selection parameters in service request; routing via SCP|D|

**Model A - Direct communication without NRF interaction**:<br>NRF와 SCP 모두 사용되지 않는다.
Consumer들은 Producer의 "NF profile"을 설정하고 그들이 선택한 producer와 직접 통신한다.

**Model B - Direct communication with NRF interaction**:<br>consumer는 NRF에 discovery를 요청한다. discovery 결과에 따라
consumer가 selection을 수행한다. consumer는 선택된 producer에게 요청을 보낸다.

**Model C - Indirect communication without delegated discovery**:<br>consumer는 NRF에 discovery를 요청한다.
discovery 결과에 따라, consumer는 NF set 또는 NF set의 특정 NF instance를 selection하고, selection된 서비스 producer의 address가 포함된 request를 SCP로 보낸다.
NF Set이 selection 된 경우, SCP가 NF service instance를 선택한다. 가능하면, SCP는 NRF를 연동하여 location, capacity 등과 같은 selection parameter를 얻는다.
SCP는 request를 선택된 NF service producer instance로 전달한다.

**Model D - Indirect communication with delegated discovery**:<br>consumer는 discovery나 selection을 하지 않는다.
consumer는 적합한 producer를 찾는 데 필요한 모든 discovery 및 selection parameter를 서비스 요청에 추가한다.
SCP는 request 메시지에서 request address와 discovery 및 selection parameter를 사용하여 request를 적절한 producer instance로 전달한다.
SCP는 NRF로 discovery를 수행하고 discovery 결과를 얻을 수 있다.

![](https://www.tech-invite.com/3m23/img/tinv-23-501-E.1-1.gif)

Ref: [_Communication models for NF/NF services interaction_](https://www.tech-invite.com/3m23/toc/tinv-3gpp-23-501_zzi.html#e-ax-E)