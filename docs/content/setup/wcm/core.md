# WCM Core opzetten

## Benodigdheden

Om onderstaande installatie te kunnen uitvoeren moet je de volgende zaken ter beschikking hebben.\
De zaken die aangegeven staan als niet verplicht zijn, zaken die je doorgaans enkel nodig hebt als je aan de core zelf wil ontwikkelen.

| Naam                    | Verplicht          |
|-------------------------|--------------------|
| Docker                  | :heavy_check_mark: |
| Digipolis VPN connectie | :heavy_check_mark: |
| NPM                     | :x:                |
| NVM                     | :x:                |

## Installatie

1. Clone de WCM Admin & WCM Gateway repositories
```bash
git clone -b master-v4 --recurse-submodules ssh://git@git.antwerpen.be/wcm/wcm-multitenancy.git
git clone ssh://git@git.antwerpen.be/wcm/wcm-gateway_service_nodejs.git
```

<!-- TODO: remove the need to checkout to another branch -->
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

    <img src="../assets/wcm-admin-mongodb-connection.png" alt="MongoDB connectie maken" width="500px"/>

    - Pas je gebruiker aan in de collectie `users` zodat de property `admin` op true staat

    <img src="../assets/wcm-admin-mongodb-set-admin.png" alt="User admin rechten geven in database" width="700px"/>

8. Ga terug naar http://localhost:3999 (of refresh) en merk op dat je gebruiker nu admin rechten heeft

De core bevat enkel de nodige informatie en tools om een WCM (tenant) context te creëren en beschikbaar te stellen aan de modules.
In de volgende stap van het installatie proces zullen de modules opgezet moeten worden.

## Tenant aanmaken
[TODO]