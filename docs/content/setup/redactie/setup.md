# Installatie en opstarten

## Benodigdheden

Om onderstaande installatie te kunnen uitvoeren moet je de volgende zaken ter beschikking hebben.\
De zaken die aangegeven staan als niet verplicht zijn, zaken die je doorgaans enkel nodig hebt als je aan de core zelf wil ontwikkelen.

| Naam                    | Verplicht           |
|-------------------------|---------------------|
| Docker                  | :fa-solid fa-check: |
| Digipolis VPN connectie | :fa-solid fa-check: |
| NPM                     | :fa-solid fa-xmark: |
| NVM                     | :fa-solid fa-xmark: |

## Installatie

1. Clone de redactie app repository
```bash
git clone ssh://git@git.antwerpen.be/reda/redactie_app_nodejs.git
```

2. Build de applicatie test
```bash
docker-compose build
```

## Opstarten

3. Start de applicatie
```bash
docker-compose up
```

## Omgevingen
De repository bevat standaard de nodige credentials om te connecteren met onze development WCM instantie.\
Hierdoor hoef je niet een WCM instantie lokaal te draaien indien je enkel wil ontwikkelen op de Redactie en niet op het WCM.

Indien je toch wil ontwikkelen tegen een lokale instantie van het WCM kan je in de `docker-compose.yml` de environment als volgt aanpassen van de `server` service:

```yaml
# ...
services:
  # ...
  server:
    env_file: local.env
    # ...
```