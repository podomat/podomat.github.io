---
title: "[Troubleshooting] Mariadb에서 sub-query에 의한 deadlock 현상"
date: 2021-09-20
categories:
  - Troubleshooting
tags:
  - DB
  - Mariadb
  - deadlock
---

얼마 전 사이트 운용실에서 vendor의 신규 장비 인수시험 중에 가상화NF의 lifecycle을 관리하는 시스템이 lifecycle operation을 수행하다가 '500 Internal Server Error'를 리턴하고 실패되는 케이스가 발생했다.