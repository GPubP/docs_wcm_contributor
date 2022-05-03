# Lokaal ontwikkelen

## Modules

Je kan gemakkelijk lokale versies van modules draaien binnen de Redactie app door de volgende stappen te doorlopen:
1. Module voorbereiden
2. Module registreren in de WCM admin (indien nog niet gebeurd)
3. Module koppelen aan lokale Redactie instantie

### Module voorbereiden

1. Clone de repo van de module
2. Volg de nodige stappen in de module om tot een build te komen\
In de meeste gevallen zijn dit de volgende stappen:\
\
Dependencies installeren:
```bash
npm i
```
Module builden:
```bash
npm run build
```

### Module registreren in WCM Admin interface

*Deze stap is enkel nodig indien de module nog niet gekend is op de WCM Admin.*

De Redactie gaat op basis van WCM Admin data modules inladen binnen een tenant.\
Hierdoor moet de module gekend en ingesteld voor de tenant waarop je aan het werken bent.

De WCM Admin interface is beschikbaar op:
  - development: https://wcm-admin-o.antwerpen.be
  - lokaal: http://localhost:3999

#### Nieuwe module registreren
1. Navigeer naar `Modules`
2. Maak een nieuwe module aan door onderaan op `New module` te klikken
3. Vul administratieve naam en beschrijving in
4. Selecteer bij module type `Frontend Bundle`
5. Vul de package naam in zoals gekend op de NPM of Digipolis Nexus registry
6. Voeg een versie toe door te klikken op `Add version`
7. Vul de gewenste versie naam in zoals gekend op de NPM of Digipolis Nexus registry
8. Voeg dependencies naar andere modules toe indien nodig
9. Klik op `confirm`
10. Klik op 'Save modules`

> [!warning]
> De module moet beschikbaar zijn met de gekozen versie voor deze te registeren.\
> Indien dit niet juist geregistreerd is, kan dit impact hebben op de installatie van andere modules.

#### Module instellen op tenant
1. Navigeer naar `Websites`
2. Bewerk de website (= tenant) waarop je wil werken tijdens ontwikkeling
3. Klap de modules sectie op door te klikken op `Show module settings`
4. Klik onderaan de sectie op `New module`
5. Selecteer je module en versie nummer
6. Klik op `Save`
7. Klik op `Save websites`

Het kan tot enkele minuten duren voor deze module effectief geïnstalleerd is.

### Module koppelen aan lokale Redactie instantie
1. Voeg een mount toe aan de `docker-compose.yml` file van de redactie app
```yaml
# ...
services:
  # ...
  server:
    volumes: 
      # ...
      - ../redactie-content_module_react:/app/server/niv_modules/redactie-content-module-1-15-1:ro
```
De volume mounting wordt als het volgt opgebouwd:\
`- [relatief pad naar module]:/app/server/niv_modules/[kebabCased module naam]-[kebabCased versie nummer]:ro`
2. run `docker-compose down` (indien er reeds een instantie draait)
3. run `docker-compose up`

> [!warning]
> De `:ro` op het einde is zeer belangrijk!\
> Dit zorgt ervoor dat je lokale module folder niet overschreven wordt.

## Core

Om gebruik te kunnen maken van de juiste linting, typescript typings, code completion en dergelijke raden we aan om ook lokaal op je machine de node_modules te installeren:
```bash
nvm use # optioneel
npm i

cd ./client
nvm use # optioneel
npm i

cd ../server
nvm use # optioneel
npm i

cd ../tenants
nvm use # optioneel
npm i
```