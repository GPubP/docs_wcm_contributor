# Hoofdstuk 3: Nieuwe WCM module opzetten
Binnen deze gids gaan we een nieuwe backend module "Greetings" opzetten op basis van de boilerplate module en deze beschikbaar stellen op onze lokale instantie van het WCM.

## Stap 1: Vertrekken van de boilerplate module

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

Eens de boilerplate lokaal binnen getrokken is, kan deze geconfigureerd, geïnstalleerd en opgestart worden.

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

Voer bovenstaande uit in zowel `./package.json` als in `./server/package.json`.

## Stap 2: Greetings BSL service installeren

We kunnen onze nieuwe bsl service builden door het volgende commando te runnen:

```bash
docker-compose build
```

Aangezien we ook gaan ontwikkelen binnen de BSL service zullen we ook lokaal de nodige modules moeten installeren (vooral voor type support):

```bash
npm i
cd ./server
npm i
```

## Stap 3: Greetings BSL environment voorbereiden
Nadat alles geïnstalleerd is, kunnen we de juiste variabelen voorzien in de configuratie van deze BSL service zodat deze straks op een correcte manier met de WCM core en andere modules kan samenwerken.

### Stap 3.1: Poort configureren
Elke service heeft zijn eigen poort die niet mag overlappen met een andere poort op je systeem. Het is dus belangrijk om in de `docker-compose.yml` file de juiste poorten te exposen. Een overzicht van alle reeds ingenomen poorten van core modules vind je [hier] (!TODO).

Voor onze module zullen we de demo module poort `60101` gebruiken.

We passen dus onze `docker-compose.yml` file als volgt aan:
```yaml
services:
  server:
    # ...
    ports:
      # App port
      - 60101:60101
      # Debug port
      - 50101:50101
```

De eerste poort wordt gebruikt door onze service zelf. De 2de poort is een debug poort en kan gebruikt worden om je lokale debugger te connecteren.

We hebben nu aan onze docker omgeving laten weten dat we een service van binnen onze container willen beschikbaar stellen op ons host machine.\
Als volgende stap zulllen we onze service zo configureren dat deze de juiste poorten gebruikt binnen de docker container.

In de `Dockerfile.dev` zorgen we ervoor dat de `--inspect` poort overeenkomt met de debug poort, ingesteld in de `docker-compose.yml` file.
```bash
...
# Start application with live reload
CMD ts-node-dev -r tsconfig-paths/register --transpileOnly --poll --inspect=0.0.0.0:50101 -- index.ts
````

### Stap 3.2: Environment variabelen
In ons scenario moeten we maar 2 environment variabelen aanpassen:
- De poort waarop onze BSL service moet luisteren
- De apikey dat de BSL service zal gebruiken zich te identificeren bij de WCM Admin.

De environment variabelen zijn te vinden in de `/server/.env/local.env` file en moeten we als volgt aanpassen:
```env
HOST=http://localhost:60101
PORT=60101

PORTAL_APIKEY=greetings-bsl-cred
```

## Stap 4: Greetings BSL module configureren in WCM Admin
Om onze BSL module binnen de WCM context te doen draaien, moeten we de volgende zaken configureren binnen de WCM Admin:
1. BSL Module registreren
2. BSL Credential aanmaken
3. BSL module registreren bij een tenant

### Stap 4.1: BSL Module registreren
De module moet eerst geregistreerd worden in de WCM Admin interface zodat deze gekend is door de gateway en andere modules.

1. Navigeer naar http://localhost:3999 en log in indien nodig.
2. Navigeer naar `Modules`.

  ![Modules overzicht in WCM Admin interface](../../../assets/wcm-admin-modules-overview.png ':size=600')

3. Klik op `New module` onderaan de pagina.
4. Vul als administratieve naam `Greetings BSL` in
5. Vul optioneel een beschrijving in
6. Selecteer bij het module type `Business service`.
7. Vul als prefix route `greetings` in.

  ![Module aanmaken in WCM Admin interface](../../../assets/greetings-module-bsl-module-configuration-1.png ':size=600')

8. Voeg een versie toe door te klikken op `Add version`.
9. Vul hier versie nummer `1.0.0` in.
10. Vul het endpoint in van de module `http://host.docker.internal:60101`
11. Voeg geen dependencies in aangezien we er voorlopig nog geen hebben.

![Module versie aanmaken in WCM Admin interface](../../../assets/greetings-module-bsl-module-version-conf.png ':size=600')

12. Klik op `confirm`.
13. Klik op `Save modules`.

### Stap 4.2: BSL Credential aanmaken
De Greetings module kan pas informatie ophalen over zijn WCM context als hij een credential (apikey) gebruikt dat gekoppeld is aan de module die we net geregistreerd hebben in de WCM Admin.

1. Navigeer naar http://localhost:3999 en log in indien nodig.
2. Navigeer naar `Server credentials`.

  ![Server credentials overzicht in WCM Admin interface](../../../assets/wcm-admin-credentials-overview.png ':size=600')

3. klik op `New server credential` onderaan de pagina.
4. Vul als administratieve naam `Greetings BSL` in.
5. Vink `Enabled` aan.
6. Selecteer bij het type `API key`.
7. Vul als apikey `greetings-bsl-cred` in (de apikey die we in de `/server/.env/local.env` file hebben voorzien).
8. Selecteer als service type `Module`.
9. Selecteer bij de Greetings BSL module.

  ![Server credentials aanmaken in WCM Admin interface](../../../assets/greetings-bsl-module-credential-conf.png ':size=600')

10. Klik op `Save credentials`.

### Stap 4.3: BSL module registreren bij een tenant
Nu de module juist geconfigureerd is kunnen we hem toevoegen aan onze tenant (= website in WCM Admin interface).

1. Navigeer naar http://localhost:3999 en log in indien nodig.
2. Navigeer naar `Websites` (= tenant).
3. Selecteer een bestaande tenant waarop je wil werken (meestal tenant-test-3).
4. Open de sectie `Modules` door te klikken op `Show module settings`.
5. Klik onderaan op `New module`.
6. Selecteer onze nieuw aangemaakte `Greetings BSL` module.
7. Selecteer versie `1.0.0`.
8. Klik op `Save`.
9. Klik op `Save website`.
 

## Stap 5: Greetings BSL module opstarten
Van zodra bovenstaande zaken in orde zijn kan je de module opstarten met het volgende commando:

```bash
docker-compose up
```

## Stap 6: Greetings BSL testen
Om te testen of de service draait, kan je surfen naar: http://localhost:60101/status. \
Als je volgend bericht krijgt, draait de server correct.

```json
{
  version: "1.0.0",
  success: true
}
```

Hiermee testen we nog niet of de service juist werkt binnen de WCM contenxt. Om dit na te gaan, moeten we de service aanspreken via de wcm-gateway.\
De wcm-gateway draait lokaal op `http://localhost:7200`. We kunnen dus testen of de WCM context weet heeft van onze nieuwe module en deze dus juist ingeladen is door de volgende call uit te voeren:

```bash
curl --location --request GET 'http://localhost:7200/proxy/admin/greetings/status'
```
Dit geeft een `401 Unauthorized` terug. Dit logisch want we hebben nog geen credential meegegeven die toegang heeft tot een tenant dat we willen aanspreken.

We moeten dus eerst onze eigen lokale credential maken.

1. Navigeer naar http://localhost:3999 en log in indien nodig.
2. Navigeer naar `Server credentials`.
3. klik op `New server credential` onderaan de pagina.
4. Vul een administratieve naam in (vrij te kiezen)
5. Vink `Enabled` aan.
6. Selecteer bij het type `API key`.
7. Vul een eigen apikey in (bv. `000-000`, maar vrij te kiezen).
8. Selecteer als service type `Consumer`.
9. Selecteer de tenant waarop onze BSL module is ingesteld (bv. tenant-test-3).
10. Klik op `Save credentials`

![Server credentials aanmaken in WCM Admin interface](../../../assets/greetings-module-personal-cred.png ':size=600')

Als we nu onze apikey meegeven bij het opvragen van de status via de gateway, krijgen we de juiste response terug:

```bash
curl --location --request GET 'http://localhost:7200/proxy/admin/greetings/status' \
--header 'apikey: 000-000'
```
```json
{
    "version": "1.0.0",
    "success": true
}
```

## FAQ

+ Ik werk op Linux en mijn service kan niet connecteren met WCM Admin +

    Windows & Mac draaien Docker in een VM. Daarom is het domein "localhost" niet hetzelfde domein als de localhost van de host machine.\
    Om alsnog via de host machine de juiste docker container aan te spreken kan er daar gebruik gemaakt worden van "host.docker.internal" in de plaats. 

    Meer informatie over "host.docker.internal" domein vind je [hier](https://docs.docker.com/desktop/windows/networking/).
    
    `host.docker.internal` werkt niet out of the box bij linux zoals dat bij Windows & Mac installaties van Docker werkt.\
    We raden aan om hier de volgende setting te voorzien in de docker-compose.yml van alle backend services die lokaal draaien:

    ```yaml
      extra_hosts:
        - "host.docker.internal:host-gateway"
    ```
