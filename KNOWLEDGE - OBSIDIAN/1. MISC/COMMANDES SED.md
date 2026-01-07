

https://quickref.me/sed.html



Delete all empty lines

```
sed '/^$/d' file.txt
```

Delete lines starting with "Hello"

```
sed '/^Hello/d' file.txt
```


Delete first line

```
sed '1d' file.txt
```


Delete last line

```
sed '$d' file.txt
```


Replace all occurrences of a string

```
sed 's/old/new/g' file.txt
```


Replace only the nth occurrence of a string

```
sed 's/old/new/2' file.txt
```


Search for a string and only print the lines that were matched

```
sed -n '/hello/p' file.txt
```

Add string to beginning of every line

```
sed 's/^/10.129.229.47 /' subdomains.txt
```

Append string to the end of every line

```sh
sed 's/$/string/' file.txt
```

Remove comments. Even those that are at the end of a line

```
sed 's/#.*$//' file.txt
```
