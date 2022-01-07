---
title: "[Shell Programming] Bash 스크립트에서 command line argument 사용하기"
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

<br>

_**<span class="order-box">1</span>Positional Parameters**_

argument가 전달된 순서에 따라 스크립트내에서 인덱스를 통해 접근할 수 있다. 첫 번째 argument는 `$1`, 두 번째 것은 `$2`와 같은 식이다. 즉, 해당 argument의 입력된 위치로 표현하는 것이다.<br>
다음 샘플 스크립트는 Username, Age, Full Name을 출력하는 스크립트이다.

#### **`userReg-positional-parameter.sh`**
```sh
echo "Username: $1";
echo "Age: $2";
echo "Full Name: $3";
```
위 스크립트에 3개의 argument를 전달하여 실행한 결과는 다음과 같다.
```
$ sh userReg-positional-parameter.sh john 25 'John Smith'
Username : john
Age: 25
Full Name: John Smith
```
<br>

_**<span class="order-box">2</span>Flags**_

flag는 argument를 스크립트에 전달하는 흔한 방법이다. 일반적으로 argument앞에 하이픈(-)으로 시작하는 flag를 사용한다.<br>
다음 샘플 스크립트는 username(-u), age(-a), full name(-f) 3개의 argument를 전달한다.<br>
`getopts` 함수가 flag를 읽고, `OPTARG`에 해당되는 value값이 읽혀진다.

#### **`userReg-flags.sh`**
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
위 스크립트에 3개의 argument를 전달하여 실행한 결과는 다음과 같다.
```
$ sh userReg-flags.sh -f 'John Smith' -a 25 -u john
Username : john
Age: 25
Full Name: John Smith
```
<br>

_**<span class="order-box">3</span>Loop Construct**_

positional parameter 방식은 유용하나 argument의 개수가 정해지지 않은 경우에는 사용하기 어렵다.
이런 경우에는 Loop Construct 방식을 고려해보는 것도 좋다.<br>
`$@`은 모든 input parameter의 배열을 의미한다. 
이것과 `for` loop를 이용하면 우리는 전달된 모든 input parameter를 순회하며 처리할 수 있다.<br>
다음 스크립트를 보자. 이것은 전달된 모든 username을 출력하는 스크립트이다.

#### **`users-loop.sh`**
```sh
i=1;
for user in "$@" 
do
    echo "Username - $i: $user";
    i=$((i + 1));
done
```

위 스크립트를 다음과 같이 실행해보자. 
argument를 몇개를 넘기든 항상 첫번째 argument부터 마지막 argument까지 순서대로 출력하게 될 것이다.

```
$ sh users-loop.sh john matt bill 'joe wicks' carol
Username - 1: john
Username - 2: matt
Username - 3: bill
Username - 4: joe wicks
Username - 5: carol
```
<br>

_**<span class="order-box">4</span>Shift Operator**_

bash에서 `shift` operator는 argument의 위치를 n개씩 왼쪽으로 이동시키고 (n을 지정하지 않으면 1이 사용됨) 가장 왼쪽에 있었던 argument는 사라지게 된다.
따라서 1씩 `shift`를 하면 매번 `$1`이 다음 argument를 가리키게 된다.<br>
`$#`은 스크립트의 input size(argument 개수)를 나타낸다. 다음 스크립트는 `$#`을 이용하여 argument들을 순회한다.

#### **`users-shift-operator.sh`**
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

위 스크립트를 실행하면 다음과 같다. 

```
$ sh users-shift-operator.sh john matt bill 'joe wicks' carol
Username - 1: john
Username - 2: matt
Username - 3: bill
Username - 4: joe wicks
Username - 5: carol
```

간단히 정리해보면,
- positional parameter는 argument의 개수와 순서를 미리 알 수 있을때 사용된다.
- flag를 사용하면 argument의 순서는 상관이 없게 된다.
- loop construct는 argument의 개수를 알 수 없을 때 편리하다.

Ref: [_How to Use Command Line Arguments in a Bash Script_](https://www.baeldung.com/linux/use-command-line-arguments-in-bash-script)