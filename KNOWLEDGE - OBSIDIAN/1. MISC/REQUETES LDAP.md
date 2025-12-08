
## Exemple de filtre

Attribut                                     OID                      Valeur
userAccountControl:   1.2.840.113556.1.4.803:    =8192


# UAC Values

![[Pasted image 20250403045616.png]]


# OID Match strings

Permet de matcher les valeurs avec les attribus

## 1. `1.2.840.113556.1.4.803`

Doit matcher entièrement

## 2. `1.2.840.113556.1.4.804`

Doit matcher si un caractère de la chaine correspond

## 3. `1.2.840.113556.1.4.1941`

Matcher les Distinguished Name d'un objet et cherche les ownership et membership entries.


# OPERATEURS LOGIQUES

& -> AND
| -> OR
! -> NOT

