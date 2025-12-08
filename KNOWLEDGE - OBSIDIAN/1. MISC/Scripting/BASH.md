

- [[#Logical Operators]]
- [[#Arguments ($0 - $9)]]
- [[#Variables]]
- [[#Arrays]]
- [[#OperateursDeComparaisonBASH]]

- [[#ArithmeticBASH]]
- [[#Input & Output]]
- [[#Boucles]]
- [[#Structures conditionnelles]]
- [[#Fonctions]]

- [[#DEBUGGINGBASH]]

## Logical Operators

| **Operator** | **Description**        |
| ------------ | ---------------------- |
| `!`          | logical negotation NOT |
| `&&`         | logical AND            |
| \|\|         | logical OR             |

Shebang #! -> indique quel interpréteur utiliser

```bash
#!/bin/bash
```

Exemple de script bash pour identifier l'IP d'un domaine

```bash
#!/bin/bash

# Check for given arguments
if [ $# -eq 0 ]
then
	echo -e "You need to specify the target domain.\n"
	echo -e "Usage:"
	echo -e "\t$0 <domain>"
	exit 1
else
	domain=$1
fi

# Identify Network range for the specified IP address(es)
function network_range {
	for ip in $ipaddr
	do
		netrange=$(whois $ip | grep "NetRange\|CIDR" | tee -a CIDR.txt)
		cidr=$(whois $ip | grep "CIDR" | awk '{print $2}')
		cidr_ips=$(prips $cidr)
		echo -e "\nNetRange for $ip:"
		echo -e "$netrange"
	done
}

# Ping discovered IP address(es)
function ping_host {
	hosts_up=0
	hosts_total=0
	
	echo -e "\nPinging host(s):"
	for host in $cidr_ips
	do
		stat=1
		while [ $stat -eq 1 ]
		do
			ping -c 2 $host > /dev/null 2>&1
			if [ $? -eq 0 ]
			then
				echo "$host is up."
				((stat--))
				((hosts_up++))
				((hosts_total++))
			else
				echo "$host is down."
				((stat--))
				((hosts_total++))
			fi
		done
	done
	
	echo -e "\n$hosts_up out of $hosts_total hosts are up."
}

# Identify IP address of the specified domain
hosts=$(host $domain | grep "has address" | cut -d" " -f4 | tee discovered_hosts.txt)

echo -e "Discovered IP address:\n$hosts\n"
ipaddr=$(host $domain | grep "has address" | cut -d" " -f4 | tr "\n" " ")

# Available options
echo -e "Additional options available:"
echo -e "\t1) Identify the corresponding network range of target domain."
echo -e "\t2) Ping discovered hosts."
echo -e "\t3) All checks."
echo -e "\t*) Exit.\n"

read -p "Select your option: " opt

case $opt in
	"1") network_range ;;
	"2") ping_host ;;
	"3") network_range && ping_host ;;
	"*") exit 0 ;;
esac
```


## Arguments ($0 - $9)

$0 -> nom du script
$1 -> arg 1
$2 -> arg 2
...
$9 -> arg 9

## Variables

Pas de types de variable à déclarer

| **Special Variable** | **Description**                                                                                                                                                         |
| -------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `$#`                 | This variable holds the number of arguments passed to the script.                                                                                                       |
| `$@`                 | This variable can be used to retrieve the list of command-line arguments.                                                                                               |
| `$n`                 | Each command-line argument can be selectively retrieved using its position. For example, the first argument is found at `$1`.                                           |
| `$$`                 | The process ID of the currently executing process.                                                                                                                      |
| `$?`                 | The exit status of the script. This variable is useful to determine a command's success. The value 0 represents successful execution, while 1 is a result of a failure. |

Ne pas mettre d'espace dans la déclaration de la variable 
(variable="blabla blabla" et pas variable = "blabla")


## Arrays

```bash
domains=(www.inlanefreight.com ftp.inlanefreight.com vpn.inlanefreight.com www2.inlanefreight.com)

echo ${domains[0]}
```


# #OperateursDeComparaisonBASH

https://tldp.org/LDP/abs/html/comparison-ops.html

### String Operators

| **Operator** | **Description**                             |
| ------------ | ------------------------------------------- |
| `==`         | is equal to                                 |
| `!=`         | is not equal to                             |
| **=~**       | match regex                                 |
| `<`          | is less than in ASCII alphabetical order    |
| `>`          | is greater than in ASCII alphabetical order |
| `-z`         | if the string is empty (null)               |
| `-n`         | if the string is not null                   |

> [!TIP]
> Penser à mettre la variable entre double-quotes ("$var")
> Si < ou > utilisé, mettre la condition entre double crochets [[ ]]
> Si =~ utilisé, mettre la condition entre double crochets

### Integer Operators

|**Operator**|**Description**|
|---|---|
|`-eq`|is equal to|
|`-ne`|is not equal to|
|`-lt`|is less than|
|`-le`|is less than or equal to|
|`-gt`|is greater than|
|`-ge`|is greater than or equal to|

> [!TIP]
> Si < ou > utilisé, mettre la condition entre double crochets [[ ]] ou double parenthèses (())

### File Operators

Vérifier l'existence / les permissions d'un fichier

| **Operator** | **Description**                                        |
| ------------ | ------------------------------------------------------ |
| `-e`         | if the file exist                                      |
| `-f`         | tests if it is a file                                  |
| `-d`         | tests if it is a directory                             |
| `-L`         | tests if it is if a symbolic link                      |
| `-N`         | checks if the file was modified after it was last read |
| `-O`         | if the current user owns the file                      |
| `-G`         | if the file’s group id matches the current user’s      |
| `-s`         | tests if the file has a size greater than 0            |
| `-r`         | tests if the file has read permission                  |
| `-w`         | tests if the file has write permission                 |
| `-x`         | tests if the file has execute permission               |


# #ArithmeticBASH

### Arithmetic Operators

> [!TIP]
> Doit être entre double parenthèses $((10 * 10))

|**Operator**|**Description**|
|---|---|
|`+`|Addition|
|`-`|Subtraction|
|`*`|Multiplication|
|`/`|Division|
|`%`|Modulus|
|`variable++`|Increase the value of the variable by 1|
|`variable--`|Decrease the value of the variable by 1|

calcul entre 2 variables

```bash
largeur=50
longueur=250

multiplication=$((longueur*largeur))

echo $multiplication
```


Calculer longueur d'une variable

```bash
echo ${#variable}
```

Afficher les 5 derniers caractères d'une variable : 

```bash
echo ${variable: -5}
```


# Input & Output

## Input #CaseBash

```bash
read -p "Select your option: " variable

```bash
case $variable in
	"1") function1 ;;
	"2") function2 ;;
	"3") function3 ;;
	"*") exit 0 ;;
esac
```

## Ouput

Possible d'envoyer l'output dans un fichier en plus de la sortie standard grâce à la commande
tee -a


# Boucles

## Boucle FOR

```bash
for variable in 1 2 3 4
do
	echo $variable
done
```

```bash
for variable in {1..30}
do
	echo $variable
done
```

## Boucle While

```bash
#!/bin/bash

counter=0

while [ $counter -lt 10 ]
do
  # Increase $counter by 1
  ((counter++))
  echo "Counter: $counter"

  if [ $counter == 2 ]
  then
    continue
  elif [ $counter == 4 ]
  then
    break
  fi
done
```

## Boucle Until 

```bash
#!/bin/bash

counter=0

until [ $counter -eq 10 ]
do
  # Increase $counter by 1
  ((counter++))
  echo "Counter: $counter"
done
```

# Structures conditionnelles

## if, elif, else

```bash
if [ $# -eq 0 ]
then
	statement
elif [ $# -eq 0 ]
then
	statement
else
	statement
fi
```
## Branches

## Case

```bash
read -p "Select your option: " variable

```bash
case $variable in
	"1") function1 ;;
	"2") function2 ;;
	"3") function3 ;;
	"*") exit 0 ;;
esac
```


# Fonctions

## Method 1 - Functions

```bash
function name {
	<commands>
}
```

## Method 2 - Functions

```bash
name() {
	<commands>
}
```

## Return Values

| **Return Code** | **Description**                |
| --------------- | ------------------------------ |
| `1`             | General errors                 |
| `2`             | Misuse of shell builtins       |
| `126`           | Command invoked cannot execute |
| `127`           | Command not found              |
| `128`           | Invalid argument to exit       |
| `128+n`         | Fatal error signal "`n`"       |
| `130`           | Script terminated by Control-C |
| `255\*`         | Exit status out of range       |
Return.sh

```bash
#!/bin/bash

function given_args {

        if [ $# -lt 1 ]
        then
                echo -e "Number of arguments: $#"
                return 1
        else
                echo -e "Number of arguments: $#"
                return 0
        fi
}

# No arguments given
given_args
echo -e "Function status code: $?\n"

# One argument given
given_args "argument"
echo -e "Function status code: $?\n"

# Pass the results of the funtion into a variable
content=$(given_args "argument")

echo -e "Content of the variable: \n\t$content"
```

# #DEBUGGINGBASH

```bash
bash -x script.sh
```

```bash
bash -x -v script.sh
```
