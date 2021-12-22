---
title: "How to Use Command Line Arguments in a Bash Script"
categories:
  - Tech
tags:
  - script
  - bash
  - command
  - argument
---

shell script 작성시 유저에게 command line argument를 입력받아서 처리하는 코드는 다음과 같이 작성할 수 있다.

**1. Positional Parameters**

Script code
```sh
echo "Username: $1";
echo "Age: $2";
echo "Full Name: $3";
```
Output
```
$ sh userReg-positional-parameter.sh john 25 'John Smith'
Username : john
Age: 25
Full Name: John Smith
```
<br>

**2. Flags**

Script code
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

Output
```
$ sh userReg-flags.sh -f 'John Smith' -a 25 -u john
Username : john
Age: 25
Full Name: John Smith
```
<br>

**3. Loop Construct**

Script code
```sh
i=1;
for user in "$@" 
do
    echo "Username - $i: $user";
    i=$((i + 1));
done
```
Output
```
$ sh users-loop.sh john matt bill 'joe wicks' carol
Username - 1: john
Username - 2: matt
Username - 3: bill
Username - 4: joe wicks
Username - 5: carol
```
<br>

**4. Shift Operator**

Script
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
Output
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