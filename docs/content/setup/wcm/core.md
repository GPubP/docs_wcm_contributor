# WCM Core opzetten

## Benodigdheden

Om onderstaande installatie te kunnen uitvoeren moet je de volgende zaken ter beschikking hebben.\
De zaken die aangegeven staan als niet verplicht zijn zaken die je doorgaans enkel nodig hebt als je aan de core zelf wil ontwikkelen.

| Naam                    | Verplicht           |
|-------------------------|---------------------|
| Docker                  | :fa-solid fa-check: |
| Digipolis VPN connectie | :fa-solid fa-check: |
| NPM                     | :fa-solid fa-xmark: |
| NVM                     | :fa-solid fa-xmark: |

## Installatie

1. Clone de WCM Admin & WCM Gateway repositories
```bash
git clone -b master-v4 --recurse-submodules ssh://git@git.antwerpen.be/wcm/wcm-multitenancy.git
git clone ssh://git@git.antwerpen.be/wcm/wcm-gateway_service_nodejs.git
```

2. Start de WCM Admin service
```bash
cd ./wcm-multitenancy
docker-compose build
docker-compose run
```

4. Start daarna de WCM gateway:
```bash
cd ./wcm-gateway_service_nodejs
docker-compose build
docker-compose run
```

5. Surf naar http://localhost:3999 en log in met een m-profiel

6. Geef je gebruiker rechten in de database (deze stap is enkel nodig voor de eerste gebruiker)\
    - Open een database connectie naar `mongodb://localhost:27009` via een MongoDB GUI (bv. [Robomongo](https://robomongo.org/))

    ![WCM Admin MongoDB configuratie](../../../assets/wcm-admin-mongodb-connection.png ':size=500')

    - Pas je gebruiker aan in de collectie `users` zodat de property `admin` op true staat

    ![Users collection updaten](../../../assets/wcm-admin-mongodb-set-admin.png ':size=700')

8. Ga terug naar http://localhost:3999 (of refresh) en merk op dat je gebruiker nu admin rechten heeft

De core bevat enkel de nodige informatie en tools om een WCM (tenant) context te creëren en beschikbaar te stellen aan de modules.
In de volgende stap van het installatie proces zullen de modules opgezet moeten worden.

## Tenant aanmaken

Een tenant aanmaken kan door te navigeren naar http://localhost:3999 en daar een nieuwe `Website` (= tenant) in te stellen.

1. Navigeer naar `Websites`.
2. Klik op `New Website`.
3. Vul een naam in dat ook bestaat op DEV.

> [!note] Users kunnen toegang krijgen tot een tenant via rechten, ingesteld op UM (User Management toepassing van Antwerpen).
> De rechten worden gechecked op basis van systeemnaam. 
> Hierdoor is het belangrijk dat de systeemnaam bij het aanmaken van een lokale tenant overeenkomt met een systeemnaam op dev.
>
> Zo kan je "meegenieten" van de UM rechten die reeds ingesteld zijn en hoef je geen eigen UM rechten laten registeren voor je lokale omgeving.

4. Vink `enabled` aan.
5. Valideer onder `Advanced` dat de `Name` overeenkomt met een `Name` op een tenant op dev (best een waar jouw gebruiker toegang toe heeft).
6. Vul als onder `Domain url` `http://localhost:3102/` in.
7. Vul onder `Database` het volgende in en laat het overige leeg:
    - Database name: `wcm-content-dev` (zelf te kiezen)
    - Database URI: `mongodb://host.docker.internal:27008`
8. Selecter onder `Modules` die je wil instellen op de tenant (zie [Modules setup](/content/setup/wcm/modules.md) indien je er geen kan toevoegen).
9. Klik op `Save website`

![Website aanmaken](../../../assets/wcm-admin-website.png ':size=900')

