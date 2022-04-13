# WCM modules opzetten

Hieronder beschrijven we de gangbare methode om een module lokaal op te zetten binnen een lokale WCM instantie.\
Het is mogelijk dat deze werkwijze in sommige modules kan afwijken.\
In dat geval kan je terecht in de readme van de module zelf.

We raden aan om onderstaande stappen door te voeren voor minstens de volgende services.\
Op basis van de startup dependency kolom kan je achterhalen in welke volgorde de services best opgestart worden.

| Naam         | Beschrijving                                   | Type   | Startup dependency        |
|--------------|------------------------------------------------|--------|---------------------------|
| WCM          | Content, Content-types & views management      | Engine | WCM Admin                 |
| Content BSL  | Content, Content-types & views management      | BSL    | WCM Admin, BraaS BSL, WCM |
| BraaS BSL    | Roles & rights                                 | BSL    | WCM Admin                 |
| Sites        | Sites management                               | Engine | WCM Admin                 |
| Sites BSL    | Sites management                               | BSL    | WCM Admin, BraaS BSL      |
| Assets       | Asset management including translation files   | Engine | WCM Admin                 |
| Assets BSL   | Asset management                               | BSL    | WCM Admin, BraaS BSL      |
| Trans BSL    | Translations service (uses assets)             | BSL    | WCM Admin, BraaS BSL      |
| Worklfow     | Allows content to follow a predefined workflow | Engine | WCM Admin                 |
| Worklfow BSL | Allows content to follow a predefined workflow | BSL    | WCM Admin, BraaS BSL      |

**Opgelet!** De modules hebben meer dependencies met elkaar dan hier weergegeven. Dit gaat enkel over de start volgorde.

## Stap 1: module installeren

1. Clone de repository
```bash
git clone [module respository url]
```
2. Build de module
```bash
docker-compose build
```

## Stap 2: module registeren
De module moet eerst geregistreerd worden in de WCM Admin interface zodat deze gekend is door de gateway en andere modules.

1. Navigeer naar http://localhost:3999 en log in indien nodig.
2. Navigeer naar `Modules`.

  ![Modules overzicht in WCM Admin interface](../../../assets/wcm-admin-modules-overview.png ':size=600')

3. Klik op `New module` onderaan de pagina.
4. Vul administratieve naam en beschrijving in.
5. Selecteer bij het module type `Business service` (of `Engine`).
6. Vul een route prefix aan zoals beschreven in de readme van de module.

  ![Module aanmaken in WCM Admin interface](../../../assets/wcm-admin-modules-general-info.png ':size=600')

7. Voeg een versie toe door te klikken op `Add version`.
8. Vul een versie nummer (meestal 1) en een beschrijving in.\
  Voorlopig wordt de versie nummer nog niet gebruikt maar in de toekomst zal dat wel het geval zijn.
9. Vul het endpoint in van de module zoals deze lokaal gehost wordt (bv. http://localhost:6021).
10. Voeg dependencies naar andere modules toe indien nodig.

  ![Module versie aanmaken in WCM Admin interface](../../../assets/wcm-admin-modules-version.png ':size=500')

11. Klik op `confirm`.
12. Klik op `Save modules`.

## Stap 3: Module credential aanmaken
De nieuwe geregistreerde module moet een credential verkrijgen waarmee hij kan communiceren met de WCM Admin.\
Deze communicatie is nodig om informatie over de tenants en andere modules te verkrijgen.


1. Navigeer naar http://localhost:3999 en log in indien nodig.
2. Navigeer naar `Server credentials`.

  ![Server credentials overzicht in WCM Admin interface](../../../assets/wcm-admin-credentials-overview.png ':size=600')

3. klik op `New server credential` onderaan de pagina.
4. Vul administratieve naam en beschrijving in.
5. Vink `Enabled` aan.
6. Selecteer bij het type `API key`.
7. Vul een apikey in dat gebruikt wordt in de config van de module.
8. Selecteer als service type `Module`.
9. Selecteer de net aangemaakte module.

  ![Server credentials aanmaken in WCM Admin interface](../../../assets/wcm-admin-credentials-create.png ':size=600')

10. Klik op `Save credentials`.

## Stap 4: module environment variabelen
1. Navigeer naar de folder van de repo.
2. Open de `/server/.env/local.env` file.
3. Pas de environment variabelen aan waar nodig (bv. `PORTAL_APIKEY` om te matchen met de server credential of `PORT`).

## Stap 5 module opstarten

Van zodra bovenstaande zaken in orde zijn kan je de module opstarten met het volgende commando:

```bash
docker-compose build
```

**Let op!** Sommige modules hebben dependencies op elkaar.\
Het is in sommige gevallen belangrijk om ze in een bepaalde volgorde op te starten.
