
# OPERATORS

- Division (`/`), Multiplication (`*`), and Modulus (`%`)
- Addition (`+`) and Subtraction (`-`)
- Comparison (`=`, `>`, `<`, `<=`, `>=`, `!=`, `LIKE`)
- NOT (`!`)
- AND (`&&`)
- OR (`||`)

# QUERY RESULTS

SORT -> ORDER BY colonne DESC/ASC;
LIMIT -> LIMIT number;
WHERE -> WHERE condition;
LIKE -> LIKE 'pattern%';



# MY #SQL / MARIADB

## Interagir avec la DB

### Login sur la db

```bash
mysql -u root -h 83.136.251.105 -P 54479 -p --ssl=false
```

### Voir user actuel

```
select user();
```

### Afficher si droits d'administration

```
SELECT super_priv FROM mysql.user WHERE user="<user>";
```

### Enumérer les DB

```sql
SHOW databases;
```

### Switch sur une DB

```sql
USE database;
```

### Enumérer les Tables de la DB

```sql
SHOW tables;
```

### Afficher les propriétés et colonnes d'une table

```sql
DESCRIBE table;
```

```sql
show columns from <table>;
```

### UNION QUERY entre la table employees et departments

```
SELECT * from employees UNION SELECT dept_no,dept_name,3,4,5,6 from departments;
```





## Backup de la DB

```
mysqldump -u root --password=root -h <hostname> -P <port> <DATABASE> > backup.sql
```

## Restauration de la DB

```
cat backup.sql | mysql -u root --password=root <DATABASE>
```


---
# MSSQL

## Client Impacket-mssqlclient

#### Login sur la db

```shell
impacket-mssqlclient -p 1433 julio@10.129.203.7 
```

windows auth

```sh
impacket-mssqlclient -p 1433 marcus@192.168.210.15 -windows-auth
```


help pour afficher les exploits possibles

#### Identifier la version de MSSQL

```sql
SELECT @@version
```

#### Afficher le user actuel

```sql
SELECT user_name()
```

#### Identifier les utilisateurs
```sql
SELECT * FROM sys.database_principals
```

#### Identifier les db

```sql
SELECT name FROM master.dbo.sysdatabases
```

#### Switch sur une db

```sql
USE flagDB
```

#### Afficher les tables de la db

```sql
SELECT table_name FROM flagDB.INFORMATION_SCHEMA.TABLES
```

#### Afficher les données de la table

```sql
SELECT * FROM tb_flag
```



---

# ORACLE Tns

## install client sqlplus

```bash
sudo apt install oracle-instantclient-sqlplus
```

## Login

```sh
sqlplus user/pass@10.129.204.235/SID
```

as sysdba

```sh
sqlplus user/pass@10.129.204.235/SID as sysdba
```

si erreur "sqlplus: error while loading shared libraries: libsqlplus.so: cannot open shared object file: No such file or directory"

```bash
sudo sh -c "echo /usr/lib/oracle/12.2/client64/lib > /etc/ld.so.conf.d/oracle-instantclient.conf";sudo ldconfig
```

## Commandes:

https://docs.oracle.com/cd/E11882_01/server.112/e41085/sqlqraa001.htm#SQLQR985

### Enumérer les tables de la DB

```bash
select table_name from all_tables;
```

### Enumérer les privilèges du user connecté

```sh
select * from user_role_privs;
```

### Enumérer les hash des passwords

```sh
select name, password from sys.user$;
```

### Bruteforce du hash: 

```bash
hashcat -a0 -m 3100 'HASH:USER' /usr/share/wordlists/rockyou.txt
```


