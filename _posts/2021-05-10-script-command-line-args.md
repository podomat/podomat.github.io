---
title: "Bash 스크립트에서 command line argument 사용하기"
last_modified_at: 2021-05-10
categories:
  - Tech
tags:
  - script
  - bash
  - command
  - argument
---

shell script 작성시 유저에게 command line argument를 입력받아서 처리하는 코드는 다음과 같이 작성할 수 있다.

> **1. Positional Parameters**

argument가 전달된 순서에 따라 스크립트내에서 인덱스를 통해 접근할 수 있다. 첫 번째 argument는 $1, 두 번째 것은 $2와 같은 식이다. 즉, 해당 argument의 입력된 위치로 표현하는 것이다.
다음 샘플 스크립트 userReg-positional-parameter.sh는 username, age, full name을 출력하는 스크립트이다.

_Script code_
```sh
echo "Username: $1";
echo "Age: $2";
echo "Full Name: $3";
```
위 스크립트를 3개의 argument를 전달하여 실행한 결과는 다음과 같다.
_Output_
```
$ sh userReg-positional-parameter.sh john 25 'John Smith'
Username : john
Age: 25
Full Name: John Smith
```
<br>

> **2. Flags**

_Script code_
```sh
while getopts u:a:f: flag
do
    case "${flag}" in
        u) username=${OPTARG};;
        a) age=${OPTARG};;
        f) fullname=${OPTARG};;
    esac
done
echo "Username: $username";
echo "Age: $age";
echo "Full Name: $fullname";
```

_Output_
```
$ sh userReg-flags.sh -f 'John Smith' -a 25 -u john
Username : john
Age: 25
Full Name: John Smith
```
<br>

> **3. Loop Construct**

_Script code_
```sh
i=1;
for user in "$@" 
do
    echo "Username - $i: $user";
    i=$((i + 1));
done
```
_Output_
```
$ sh users-loop.sh john matt bill 'joe wicks' carol
Username - 1: john
Username - 2: matt
Username - 3: bill
Username - 4: joe wicks
Username - 5: carol
```
<br>

> **4. Shift Operator**

_Script code_
```sh
i=1;
j=$#;
while [ $i -le $j ] 
do
    echo "Username - $i: $1";
    i=$((i + 1));
    shift 1;
done
```
_Output_
```
$ sh users-shift-operator.sh john matt bill 'joe wicks' carol
Username - 1: john
Username - 2: matt
Username - 3: bill
Username - 4: joe wicks
Username - 5: carol
```
<br>

Ref: [_How to Use Command Line Arguments in a Bash Script_](https://www.baeldung.com/linux/use-command-line-arguments-in-bash-script)