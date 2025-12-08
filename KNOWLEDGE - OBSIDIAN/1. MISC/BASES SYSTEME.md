

# KERNEL

Fait la liaison entre la couche matérielle et logicielle.

![[Pasted image 20250329231337.png]]


# Espace Kernel/User

> [!TIP]
> Pour qu'un processus en Userland puisse interagir avec le matériel (support de stockage, clavier, souris, écran...), celui-ci fait appel aux API système qui effectuent un SYSCALL vers le KERNEL

## User land

- Espace mémoire attribué aux processus utilisateurs
- Processus à faible priorité

## Kernel land

- Espace mémoire utilisé par les fonctions système critiques
- Exécution des pilotes de périphériques
- Structure de données et objets systèmes

![[Pasted image 20250329232634.png]]


---
# API & SYSCALLS


![[Pasted image 20250329234237.png]]

---
# PROCESSUS

Conteneur virtuel contenant:
- Zones en lecture/ecriture pour le stockage de données temporaires (Heap(Tas) et Stack(Pile))
- Le Portable Executable (Headers et Sections)
- Les librairies nécessaires au fonctionnement
- Les threads du programme
- Les informations du processus
- La structure de données reservées au système

![[Pasted image 20250329234920.png]]

---
# SEGMENTATION MEMOIRE INTER-PROCESSUS

- Chaque processus est isolé sur le système (Il se croit seul)
- Il ne récupère pas d'espace mémoire physique directement
- Il est sandboxé avec une plage de mémoire virtuelle allouée (4Go pour les 32bits)
- Le kernel intervient pour faire l'intermédiaire entre les plages de mémoire vituelle et physique au travers d'une table de pagination

![[Pasted image 20250330004806.png]]

---
# MEMOIRE

## Buffer

![[Pasted image 20250522225000.png]]

![[Pasted image 20250329220419.png]]

## HEAP (TAS)

Présent dans la RAM.
Stocke les variables globales.


## STACK (PILE)

Présent dans la RAM.
- Stocke les variables déclarées dans les fonctions. 
- LIFO (Last In First Out)


---
# CPU REGISTERS


- Elément essentiel d'un CPU.
- Offre un petit espace mémoire pour stocker des données temporaires.
- General registers, Control registers, and Segment registers


## General registers

#### Data registers

|**32-bit Register**|**64-bit Register**|**Description**|
|---|---|---|
|`EAX`|`RAX`|Accumulator is used in input/output and for arithmetic operations|
|`EBX`|`RBX`|Base is used in indexed addressing|
|`ECX`|`RCX`|Counter is used to rotate instructions and count loops|
|`EDX`|`RDX`|Data is used for I/O and in arithmetic operations for multiply and divide operations involving large values|

#### Pointer registers

| **32-bit Register** | **64-bit Register** | **Description**                                                                                                                         |
| ------------------- | ------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| `EIP`               | `RIP`               | Instruction Pointer stores the offset address of the next instruction to be executed (Adresse mémoire du jeu d'instructions à éxecuter) |
| `ESP`               | `RSP`               | Stack Pointer points to the top of the stack (Fin de la stack frame)                                                                    |
| `EBP`               | `RBP`               | Base Pointer is also known as `Stack Base Pointer` or `Frame Pointer` thats points to the base of the stack (Début de la stack frame)   |

#### Index registers

|**Register 32-bit**|**Register 64-bit**|**Description**|
|---|---|---|
|`ESI`|`RSI`|Source Index is used as a pointer from a source for string operations|
|`EDI`|`RDI`|Destination is used as a pointer to a destination for string operations|


## Stack Frames

#### Prologue

```shell-session
(gdb) disas bowfunc 

Dump of assembler code for function bowfunc:
   0x0000054d <+0>:	    push   ebp       # <---- 1. Stores previous EBP
   0x0000054e <+1>:	    mov    ebp,esp   # <---- 2. Creates new Stack Frame
   0x00000550 <+3>:	    push   ebx
   0x00000551 <+4>:	    sub    esp,0x404 # <---- 3. Moves ESP to the top
   <...SNIP...>
   0x00000580 <+51>:	leave  
   0x00000581 <+52>:	ret    
```

#### Epilogue

```shell-session
(gdb) disas bowfunc 

Dump of assembler code for function bowfunc:
   0x0000054d <+0>:	    push   ebp       
   0x0000054e <+1>:	    mov    ebp,esp   
   0x00000550 <+3>:	    push   ebx
   0x00000551 <+4>:	    sub    esp,0x404 
   <...SNIP...>
   0x00000580 <+51>:	leave  # <----------------------
   0x00000581 <+52>:	ret    # <--- Leave stack frame
```


## Endianness

### Big Endian / Little Endian

Now, let us look at an example with the following values:

- Address: `0xffff0000`
- Word: `\xAA\xBB\xCC\xDD`

|**Memory Address**|**0xffff0000**|**0xffff0001**|**0xffff0002**|**0xffff0003**|
|---|---|---|---|---|
|Big-Endian|AA|BB|CC|DD|
|Little-Endian|DD|CC|BB|AA|