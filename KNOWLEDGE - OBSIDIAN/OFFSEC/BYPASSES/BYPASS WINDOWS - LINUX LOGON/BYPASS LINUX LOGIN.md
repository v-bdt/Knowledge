

# GRUB

1. Boot sur le grub
2. Se positionner sur les options avancées et appuyer sur E
3. Trouver la ligne "linux /boot/vmlinuz[...] ro quiet"
4. Remplacer "ro quiet" par "rw init=/bin/bash"
5. Appuyer sur F10
6. poweroff -f pour sortir proprement et éviter le kernel panick

