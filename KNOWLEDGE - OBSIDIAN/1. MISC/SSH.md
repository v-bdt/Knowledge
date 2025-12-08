
Générer clé SSH

```bash
ssh-keygen -f key
```

Ajouter clé publique aux "authorized keys"

```bash
cat key.pub >> /root/.ssh/authorized_keys
```

```bash
echo "ssh-rsa AAAAB...SNIP...M= user@parrot" >> /root/.ssh/authorized_keys
```

Se connecter avec clé privé:

```bash
chmod 600 key
```

```bash
ssh root@10.10.10.10 -i key
```