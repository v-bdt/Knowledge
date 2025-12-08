
## Regex

Recherche mot finissant en .txt sur ligne contenant le mot documents et enlève les caractères ' " <>
```bash
grep -oP "/documents[^'\"<>]*?\.txt"
```