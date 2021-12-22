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

**Positional Parameters**

Script
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

**Flags**

Script
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

**Loop Construct**

Script
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

**Shift Operator**

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

원문: [_How to Use Command Line Arguments in a Bash Script_](https://www.baeldung.com/linux/use-command-line-arguments-in-bash-script)