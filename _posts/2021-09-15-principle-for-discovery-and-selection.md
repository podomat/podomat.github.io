---
title: "NF 및 NF service의 discovery와 selection"
date: 2021-09-15
categories:
  - Tech
tags:
  - 5G
  - SCP
---

### General

NF 또는 SCP는 연동할 특정 타입 NF의 Service Instance를 찾기 위해 NRF가 제공하는 `NF/NF Service Discovery`를 이용한다.

**NOTE** NRF는 SCP와 함께 위치할 수 있다.
{: .notice}

NRF가 discovery 서비스를 제공하기 위해서는 `NF Profile`이 NRF에 등록되어 있어야 한다.
`NF Profile`은 NRF가 (`NF/NF Service Instance`가 처음 동작할 때 같은 경우에) `NF Profile`이 포함된 NFRegister 메시지를 받아서 등록한다.

NF 또는 SCP가 등록된 `NF/NF Service`에 대한 정보를 얻기 위해 원하는 `NF type` 정보를 전달하고, 추가로 특정 서비스에 대한 목록이나 slice 정보 등을 제공할 수 있다.
또한, `NF Set` 관련 정보를 제공하여 `NF Set` 내에서 NF Instance를 re-selection 할 수 있게 한다.

가입자 데이터에 접근하는 NF(ex, HSS, UDM)의 경우, NRF는 제공된 가입자ID에 해당하는 `NF Group ID`를 확인할 수 있어야 한다.
NRF가 이 정보(가입자ID에 대한 NF그룹ID의 매핑)를 가지고 있지 않다면,
UDR이 제공하는 Nudr_GroupIDmap_Query 서비스를 이용하여 가입자ID에 해당하는 NF Group ID를 조회할 수 있다.

Indirect Communication 방식에서 NF는 Request 메시지를 타겟에 전달하기 위해 SCP를 이용한다.
Delegate Discovery 모드로 설정된 NF는 discovery 절차를 생략하고 SCP에 이 절차를 위임한다.
이때, NF는 SCP가 discovery 및 selection 하는데 필요한 파라미터를 Request 메시지에 포함한다.
SCP는 NRF를 연동하여 discovery 결과를 얻고, NRF나 UDR을 연동하여 가입자ID에 해당하는 NF Group ID를 얻는다.