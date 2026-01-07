
Scan repo

```bash
git log -p | scanrepo
```

afficher commit

```bash
git show <commit>
```

---
# Gestion repo SSH

Mettre en place clé SSH pour gérer repo -> https://dev.to/hbolajraf/git-connecting-to-github-and-pushing-changes-using-ssh-on-windows-2f5

0. Vérifier qu'on est connecté au serveur

```sh
ssh -vT git@github.com
```

1. Ajouter clé ssh si nécessaire

```sh
ssh-add ~/.ssh/id_rsa
```

2. Mise a jour du repo en faisant un Pull

```sh
git pull origin main
```

2. Afficher les modifs 

```sh
git status
```

3. Ajout des fichiers modifiés

```sh
git add .
```

4. Afficher les modifs 

```sh
git status
```

5. Commit pour tracer les changements effectués

```sh
git commit -m "Your commit message"
```

4. Pousser les changements sur le repo

```sh
git push origin main
```
