
- [[#FRAMEWORKS]]
- [[#APIs]]

# FRAMEWORKS

- [Laravel](https://laravel.com/) / Symfony (`PHP`): usually used by startups and smaller companies, as it is powerful yet easy to develop for.
- [Express](https://expressjs.com/) (`Node.JS`): used by `PayPal`, `Yahoo`, `Uber`, `IBM`, and `MySpace`.
- [Django](https://www.djangoproject.com/) / Flask (`Python`): used by `Google`, `YouTube`, `Instagram`, `Mozilla`, and `Pinterest`.
- [Rails](https://rubyonrails.org/) (`Ruby`): used by `GitHub`, `Hulu`, `Twitch`, `Airbnb`, and even `Twitter` in the past

## Templates

JINJA2 -> FLASK / DJANGO
TWIG -> Symfony

Exploiter [[SSTI (Server-Side Template Injection)]]

---
# APIs

Standards d'API -> SOAP, REST, GRAPHQL & gRPC

- [[#SOAP]]
- [[#REST]]
- [[#EXPLOITATION Web Services & API]]


## SOAP

Data au format XML et envoyé au travers de requêtes HTTP

```xml
<?xml version="1.0"?>

<soap:Envelope
xmlns:soap="http://www.example.com/soap/soap/"
soap:encodingStyle="http://www.w3.org/soap/soap-encoding">

<soap:Header>
</soap:Header>

<soap:Body>
  <soap:Fault>
  </soap:Fault>
</soap:Body>

</soap:Envelope>
```
## REST

Data au format JSON et envoyé au travers de d'endpoints accessibles depuis l'URL (ex: search/users/1)

```json
{
  "100001": {
    "date": "01-01-2021",
    "content": "Welcome to this web application."
  },
  "100002": {
    "date": "02-01-2021",
    "content": "This is the first post on this web app."
  },
  "100003": {
    "date": "02-01-2021",
    "content": "Reminder: Tomorrow is the ..."
  }
}
```

# EXPLOITATION [[Web Services & API]]




