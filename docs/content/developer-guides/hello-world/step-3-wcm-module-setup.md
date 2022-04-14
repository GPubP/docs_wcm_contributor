## Hoofstuk 3: Nieuwe WCM module opzetten
Binnen deze gids gaan we een nieuwe backend module "Greetings" opzetten op basis van de boilerplate module en deze beschikbaar stellen op onze lokale instantie van de WCM.

## Vertrekken van de boilerplate module

Bij het opzetten van een nieuwe module raden we aan om altijd te beginnen van de [boilerplate bsl module](https://bitbucket.antwerpen.be/projects/WCM/repos/wcm-boilerplate-bsl_service_nodejs/browse).

```bash
# Clone de boilerplate code
git clone --depth=1 --branch=master ssh://git@git.antwerpen.be/wcm/wcm-boilerplate-bsl_service_nodejs.git

# verwijder alle referenties naar de boilerplate git
rm -rf ./wcm-boilerplate-bsl_service_nodejs/.git

# Hernoem de folder naar Greetings [eigen-naam] (zie verder) module 
mv ./wcm-boilerplate-bsl_service_nodejs ./wcm-greetings-bsl_service_nodejs
cd ./wcm-greetings-bsl_service_nodejs
```

Eens we de boilerplate lokaal binnen getrokken hebt, kunnen we deze configureren, installeren en opstarten.

Als eerste stap moet de package.json opnieuw geconfigureerd worden zodat hier geen verwijzingen meer staan naar de "boilerplate".
```json
{
  "name": "wcm-greetings-shd-bsl_service_nodejs",
  "version": "1.0.0",
  "description": "Greetings SHD BSL",
  "homepage": "<your git repository link or homepage link>",
   "repository": {
    "type": "git",
    "url": "git+<your git repository link>"
  },
  "bugs": {
    "url": "<link to issues page>"
  },
  ...
}
```

Om niet met meerderen dezelfde module aan te maken kunnen we best de `[naam/bedrijfsnaam]` aanpassen naar ons eigen naam of bedrijf (bv. `@redactie/greetings-shd-module`)

## Greetings BSL service installeren

We kunnen onze nieuwe bsl service builden door het volgende commando te runnen:

```bash
docker-compose build
```

Aangezien we ook gaan ontwikkelen binnen de BSL service moeten we lokaal de nodige modules installeren (vooral voor type support):

```bash
npm i
cd ./server
npm i
```



1. Clone & package.json
2. Install & Build
3. Registreren in WCM Admin
4. Run
