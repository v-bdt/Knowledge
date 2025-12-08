

Convertir SID en Hexa au format S-1-5... #SID

```sh
python3 -c "import sys;h='<HEX_VALUE>';h=h[2:] if h.startswith(('0x','0X')) else h;d=bytes.fromhex(h);r=d[0];c=d[1];ia=int.from_bytes(d[2:8],'big');subs=[int.from_bytes(d[8+4*i:12+4*i],'little') for i in range(c)];print(f'S-{r}-{ia}-'+'-'.join(map(str,subs)))"
```