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

**NOTE:** NRF는 SCP와 함께 위치할 수 있다.
{: .notice}

NRF가 discovery 서비스를 제공하기 위해서는 `NF Profile`이 NRF에 등록되어 있어야 한다.
`NF Profile`은 NRF가 (NF/NF Service Instance가 처음 동작할 때 같은 경우에) `NF Profile`이 포함된 NFRegister 메시지를 받아서 등록한다.

NF 또는 SCP가 등록된 `NF/NF Service`에 대한 정보를 얻기 위해 원하는 `NF type` 정보를 전달하고, 추가로 특정 서비스에 대한 목록이나 slice 정보 등을 제공할 수 있다.
또한, `NF Set` 관련 정보를 제공하여 `NF Set` 내에서 NF Instance를 re-selection 할 수 있게 한다.

가입자 데이터에 접근하는 NF(ex, HSS, UDM)의 경우, NRF는 제공된 가입자ID에 해당하는 `NF Group ID`를 확인할 수 있어야 한다.
NRF가 이 정보(가입자ID에 대한 NF그룹ID의 매핑)를 가지고 있지 않다면,
UDR이 제공하는 `Nudr_GroupIDmap_Query` 서비스를 이용하여 가입자ID에 해당하는 `NF Group ID`를 조회할 수 있다.

Indirect Communication 방식에서 NF는 Request 메시지를 타겟에 전달하기 위해 SCP를 이용한다.
Delegate Discovery 모드로 설정된 NF는 discovery 절차를 생략하고 SCP에 이 절차를 위임한다.
이때, NF는 SCP가 discovery 및 selection 하는데 필요한 파라미터를 Request 메시지에 포함한다.
SCP는 NRF를 연동하여 discovery 결과를 얻고, NRF나 UDR을 연동하여 가입자ID에 해당하는 `NF Group ID`를 얻는다.

**NOTE**: SCP는 HSS/UDM에 대한 delegated discovery 시에 NRF에게 사용자 ID를 전달하고 HSS/UDM 인스턴스 그룹 정보를 찾거나
아니면 UDR에 HSS/UDM Group ID를 질의할 수 있다.
{: .notice}

NRF는 discovery 조건에 해당하는 NF/NF Service Instance 목록을 제공하고,
NF Instance의 IP주소나 FQDN, NF Service Instance와 관련된 endpoint 주소들을 NF나 SCP에게 제공한다.
그리고, `NF Set ID`나 `NF Service Set ID`도 제공한다.
discovery 응답에는 제공된 discovery 결과가 캐쉬될 수 있도록 유효기간을 포함한다.
유효기간 동안에는 조건이 같다면 discovery 결과도 어떤 가입자에게나 동일하므로,
discovery 응답으로 수신한 `NF Profile`들의 정보는 캐쉬되었다가 NF selection에 사용될 수 있다.

Direct Communication 방식에서 NF는 discovery 결과를 이용하여 NF/NF Service Instance를 선택한다.

Delegated Discovery가 없는 Indirect Communication 방식에서 NF는 discovery 결과를 사용하여 NF Instance를 선택하는 반면,
연관된 NF Service Instance의 선택은 NF 또는 SCP에 의해 수행될 수 있다.

위의 두 경우 모두에서 NF는 subsequent selection을 위해 캐쉬된 discovery 결과를 사용할 수 있다
즉, NF가 selection을 위해 discovery 절차를 다시 수행할 필요가 없다.

Delegated Discovery를 사용한 Indirect Communication 방식에서
SCP는 NF가 제공한 discovery 및 selection 파라미터를 기반으로 필요하다면 NRF와 연동하여 적절한 NF/NF Service Instance를 검색하고 선택한다.
사용될 NRF는 NF Consumer가 discovery 파라미터의 일부로 제공할 수 있다. (NSSF 쿼리의 결과 등으로 얻어짐)
SCP는 subsequent selection을 위해 캐쉬된 discovery 결과를 사용할 수 있다.
즉, SCP가 selection을 수행하기 위해 discovery 절차를 다시 수행할 필요가 없다.

**NOTE**: 하나의 PLMN에 Direct/Indirect Communication을 각각 또는 모두 적용할 수 있다.
{: .notice}

NF 또는 SCP는 NF Profile의 변경(ex, NF Service Instance가 서비스를 시작 또는 중지함) 또는
새로 등록 및 등록 취소 된 NF Instance에 대한 notification을 수신하기 위해 NRF에 구독 할 수 있다.

여러 PLMN에 걸친 NF/NF Service Discovery의 경우, 로컬 PLMN의 NRF는 원격 PLMN의 NRF와 연동하여
discovery 조건에 맞는 원격 PLMN의 NF Instance의 NF Profile을 검색한다.
로컬 PLMN의 NRF는 NF가 제공한 PLMN ID를 사용하여 타겟 PLMN 특정 쿼리를 만들어서 원격 PLMN의 NRF에 전달한다.

