# Redactie setup

De Redactie is een modulaire interface op de WCM API.\
Voor lokale ontwikkeling (op Dev WCM API) is het voldoende om enkel de redactie app te draaien.

## Redactie lokaal opzetten
Indien je de Redactie app lokaal wil draaien kan je de volgende stappen uitvoeren:

```bash
git clone ssh://git@git.antwerpen.be/reda/redactie_app_nodejs.git
```
```bash
docker-compose build
```
```bash
docker-compose up
```

Meer informatie over deze stappen vind je hier: [Installeren en opstarten](/content/setup/redactie/setup.md)

## Ontwikkelen op de Redactie
Indien je aanpassingen wil doen op de Redactie core of een van zijn modules kan de volgende stappen ondernemen:

[Lokaal ontwikkelen](/content/setup/redactie/dev-setup.md)
