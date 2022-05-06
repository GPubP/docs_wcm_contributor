# Hoofdstuk 1: Nieuwe Redactie module opzetten

Binnen deze gids gaan we een nieuwe frontend module "Greetings" opzetten op basis van de boilerplate module en deze beschikbaar stellen op onze lokale instantie van de Redactie app.

## Stap 1: Vertrekken van de boilerplate module

Bij het opzetten van een nieuwe module raden we aan om altijd te beginnen van de [boilerplate module](https://bitbucket.antwerpen.be/projects/REDA/repos/redactie-boilerplate_module/browse).

```bash
# Clone de boilerplate code
git clone --depth=1 --branch=master ssh://git@git.antwerpen.be/reda/redactie-boilerplate_module.git

# verwijder alle referenties naar de boilerplate git
rm -rf ./redactie-boilerplate_module/.git

# Hernoem de folder naar Greetings [eigen-naam] (zie verder) module 
mv ./redactie-boilerplate_module ./redactie-greetings-shd_module_react
cd ./redactie-greetings-shd_module_react
```

Eens we de boilerplate lokaal binnen getrokken hebben, kunnen we deze configureren, installeren en publiceren.

Als eerste stap moet de package.json opnieuw geconfigureerd worden zodat hier geen verwijzingen meer staan naar de "boilerplate".
```json
{
  "name": "@redactie/greetings-[naam/bedrijfsnaam]-module",
  "version": "0.0.0-1",
  "description": "Redactie greetings module",
  "main": "dist/redactie-greetings-module.umd.js",
  "module": "dist/redactie-greetings-module.umd.js",
  "types": "dist/index.d.ts",
  "jsnext:main": "<dist/redactie-greetings-module.umd.js>",
  "repository": {
    "type": "git",
    "url": "<your git repository link>"
  },
  "keywords": [
    "Redactie",
    "React",
    "Greetings",
    "Module"
  ],
```

Om niet met meerdere partijen dezelfde module aan te maken kunnen we best de `[naam/bedrijfsnaam]` aanpassen naar ons eigen naam of bedrijf (bv. `@redactie/greetings-shd-module`)

## Stap 2: Greetings module installeren & publishen

Daarna moet de module een eerste maal geïnstalleerd, gebuild en gepubliceerd worden.

```bash
npm i

npm run build

npm version patch

# De `publishConfig` in de package.json file zal ervoor zorgen dat de module gepublished wordt op de Digipolis Nexus
npm publish
```

Het publiceren van een initiële versie is nodig om op deze module te ontwikkelen.\
Er moet een versie op de Digipolis Nexus repository of main NPM repository staan om deze te kunnen inladen binnen de Redactie app.

## Stap 3: Greetings module configureren in WCM Admin

Van zodra de module beschikbaar is in de registry, kunnen we deze [module configureren in de WCM Admin interface](/content/setup/redactie/dev-setup?id=module-registreren-in-wcm-admin-interface).

Maak een nieuwe module aan en stel deze als volgt in:

![Hello world module aanmaken](../../../assets/hello-world-module-main-setup.png ':size=700')

Voeg daarna een versie aan deze module toe met de volgende waarden en bewaar de module

![Hello world module aanmaken](../../../assets/hello-world-module-version-setup.png ':size=500')

Als laatste moet de nieuw geregistreerde module nog [ingesteld worden op een tenant](/content/setup/redactie/dev-setup?id=module-instellen-op-tenant)

## Stap 4: Module lokaal mounten

De module is nu ingesteld op de tenant en zal al doorkomen als we op onze lokale instantie van de redactie app navigeert naar de tenant.

<!-- TODO: afbeelding toevoegen van tenant -->
[TODO: afbeelding]

De versie die momenteel ingeladen wordt is de versie die we net gepubiceerd hebben in een NPM registry.\
Om effectief lokaal te onwikkelen moeten we deze module lokaal overschrijven.\
Dit kan door een mount in te stellen in de `docker-compose.yml` file van de Redactie app.

```yaml
# ...
services:
# ...
server:
 volumes: 
   # ...
   - ../redactie-greetings-shd_module_react:/app/server/niv_modules/redactie-greetings-shd-module-0-0-1:ro
```

Om deze nieuwe configuratie door te voeren, moet de redactie app herstart worden:

```bash
docker-compose down && docker-compose up
```

Meer info over modules lokaal mounten vind je [hier](/content/setup/redactie/dev-setup?id=module-koppelen-aan-lokale-redactie-instantie).

## Stap 5: Testen

We hebben nu alle nodige stappen ondernomen om de module lokaal werkende te krijgen.\
We kunnen dit nu gaan valideren door een aanpassing te doen in onze lokale versie van de module en dan na te gaan of deze aanpassing doorkomt.

Als eerste stap moeten we de watcher opstarten zodat de module opnieuw gebuild wordt bij elke wijziging die we doen:
```bash
npm run build:w
```

Voeg daarna de volgende log toe aan de `public/index.tsx` file van de greetings module:
```ts
import './lib/api';
import './lib/routes';
import { registerTranslations } from './lib/i18next';

/* ------- Onderstaande lijn toevoegen aan de file ------- */
console.log('lokale versie van de greetings module geladen');

registerTranslations();

/**
 * @module Exports
 */
// Export types needed for other modules here
export * from './lib/boilerplate.types';
```

Navigeer naar [https://localhost:3002](https://localhost:3002) en selecteer je tenant. Open de console en valideer of de log doorkomt (zie [FAQ](#faq) indien dit niet het geval is).\
We zijn nu klaar om onze eerste pagina te bouwen!

### FAQ

+ Ik krijg geen log te zien +

  Indien alle stappen correct verlopen zijn en je nog steeds geen log ziet, kan je de volgende zaken proberen:
  - Zorg ervoor dat je de pagina inlaadt zonder cache (Tip: zet je console open en selecteer daar `Disable cache` in de network tab).
  - Valideer of het lokale pad (deel voor `:`) in de `docker-compose.yml` het juiste relatief pad bevat naar je lokale module folder.
  - Ga na of de module juist geconfigureerd is in de WCM Admin en of deze correct gekoppeld is aan de tenant.

+ Ik kan niet publiceren +

  Standaard verwijst de boilerplate publicatie configuratie naar de Digipolis Nexus Repository. Dit is zo goed als altijd het gewenste gedrag.

  Om te kunnen publiceren naar deze Repository moet je een ex account hebben en via [UM](https://um.antwerpen.be) `developer` rechten krijgen tot de Nexus Repository. 
  De verwerking van deze aanvraag kan tot 2u duren.

  Hierna moet je inloggen op deze repository via het volgende commando:

  ```bash
  npm adduser --registry="https://nexusrepo.antwerpen.be/repository/npm-private"
  ```

  NPM zal je vragen naar een `username`, `paswoord` en `e-mailadres`.
  
  De username en het paswoord komen overeen met je ex nummer.\
  Het e-mailadres mag je zelf kiezen maar we raden aan om hiervoor je digipolis e-mailadres te gebruiken.

+ Ik heb geen toegang tot de WCM Admin op develoment +

  Indien je nog geen toegang hebt tot de WCM Admin interface moet je eerst even een inlog poging doen.\
  Daarna stuur je een e-mail naar de Digipolis service desk met de vraag om jezelf toegang te geven (<a href="mailto:servicedesk@digipolis.be" alt="Digipolis service desk">servicedesk@digipolis.be</a>)
