---
title: "NF service 연동을 위한 Communication model"
date: 2021-09-10
categories:
  - Tech
tags:
  - 5G
  - SCP
---

|Communication between consumer and producer|Service discovery and request routing|Communication model|
|---|---|:---:|
|Direct communication|No NRF or SCP; direct routing|A|
|Direct communication|Discovery using NRF services; no SCP; direct routing|B|
|Indirect communication|Discovery using NRF services; selection for specific instance from the Set can be delegated to SCP. Routing via SCP|C|
|Indirect communication|Discovery and associated selection delegated to an SCP using discovery and selection parameters in service request; routing via SCP|D|





Ref: [_Communication models for NF/NF services interaction_](https://www.tech-invite.com/3m23/toc/tinv-3gpp-23-501_zzi.html#e-ax-E)