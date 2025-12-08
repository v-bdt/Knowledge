
> [!TIP]
> Permet d'obtenir une liste d'hotes dans le domaine et énumère les partages et répertoires accessibles en lectures sur ceux ci. Il récupère ensuite tous les fichiers accessibles.
> Doit être lancé dans un contexte utilisateur du domaine ou ordinateur du domaine



```powershell
Snaffler.exe -s -d inlanefreight.local -o snaffler.log -v data
```