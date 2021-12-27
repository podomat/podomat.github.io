---
title: "SVN에서 외부 Differencing Tool 사용하기"
last_modified_at: 2021-05-16
date: 2021-05-15
categories:
  - Tech
tags:
  - version control
  - subversion
  - svn
  - diff
  - vimdiff
---
<br>
_**<span class="order-box">1</span>SVN diff**_

**subversion**에서 버전간 상세 변경 내역을 확인하는 명령어로 `svn diff`가 있다.
이것을 수행하면 GNU diff의 출력 포맷으로 비교 결과를 보여준다.
```
%  svn diff evt_main.c 
Index: evt_main.c
===================================================================
--- evt_main.c	(리비전 453)
+++ evt_main.c	(작업 사본)
@@ -391,7 +391,7 @@
 	mplog_t* log_hdl = vnfmlog_init(getenv("SYSTEM_TARGET_HOME"), getenv("SCRIPT_FILENAME"));
 	MYSQL* db_conn = vnfm_construct_db(log_hdl, getenv("SYSTEM_TARGET_HOME"), VNFM_DB);
 	VnfEventType event_type = VNF_EVT_TYPE_UNKNOWN;
-	char vnfa_auth_token[] = "0325a6c9fb9f986635f341fe1fc164f5c2377c5b9eee471a72691c4c";
+	char vnfa_auth_token[] = "0325a6c9fb9f986635f341fe1fc164f5c2377c5b9eee471a72691c4c"; // 대학교 코딩 동아리...?
 	char user_name[VIMS_MAX_NORMAL_STRING_128] = {0};
 	char user_id[VIMS_MAX_NORMAL_STRING_32] = {0};
```
나에게 이것은 결코 보기 편한 형태라고 얘기하긴 어렵다.
subversion에서 제공하는 external diff 기능과 vimdiff를 조합하여 다음과 같이 좀 더 쉽게 알아볼 수 있는 형태로 만들어 보자.
![img](/assets/images/posts/svn-vimdiff-output.jpg?raw=true)
<br><br>

_**<span class="order-box">2</span>SVN diff with vimdiff**_

vimdiff을 external diff으로 지정하기 위해서는 먼저 다음과 같이 wrapping script를 작성해준다.

***diffwrap.sh***
```sh
#!/bin/sh

# Configure your favorite diff program here.
DIFF="/bin/vimdiff"

# Subversion provides the paths we need as the sixth and seventh 
# parameters.
LEFT=${6}
RIGHT=${7}

# Call the diff command (change the following line to make sense for
# your merge program).
$DIFF $LEFT $RIGHT

# Return an errorcode of 0 if no differences were detected, 1 if some were.
# Any other errorcode will be treated as fatal.
```
작성한 script를 다음과 같이 `svn diff` 커맨드에 옵션으로 추가하면 vimdiff의 형태로 출력된다.
```sh
% svn diff --diff-cmd=diffwrap.sh ${filename}
```
하지만, 매번 위와 같이 `--diff-cmd`를 입력하는건 불편하다. 좀 더 편하게 만들어 보자.<br>
${home}/.subversion/config 파일의 `diff-cmd` 항목에 다음과 같이 script path를 설정한다.
```sh
...
diff-cmd = ${script-fullpath}/diffwrap.sh
...
```
이제는 `svn diff` 실행시에 `--diff-cmd`옵션을 입력하지 않아도 적용된다.
<br><br>

Ref: [_Using External Differencing Tools_](https://svnbook.red-bean.com/en/1.4/svn.advanced.externaldifftools.html)
