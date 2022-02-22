## Enkele basis begrippen

>[!TIP|icon:fas fa-forward|label:Skip dit hoofdstuk]
> Weet je reeds hoe basis REST API's werken, ga dan ineens verder naar het [API ontwerp](/content/designers/design) of toch nog eerst even nalezen hoe je een voorbereidende [functionele analyse](/content/designers/analysis) maakt.

### Request en Response

Misschien triviaal, maar HTTP, de basis waarop onze REST API's zijn gebaseerd, is in essentie een [Request/Response](https://en.wikipedia.org/wiki/Request%E2%80%93response) model.

Tussen 2 partijen is er een eerste die iets vraagt en een tweede die iets antwoordt. Soms zeggen we wel dat een consumer iets vraagt aan een provider of ook wel eens een client die aan een server iets vraagt en een antwoord terugkrijgt.

Hier komen we verder op terug.

### Think Resources

De basis van [REST](https://en.wikipedia.org/wiki/Representational_state_transfer) API's zijn [Resources](https://github.com/digipolisantwerpdocumentation/api-system#rest-introductie). Dit is een abstract concept en is in essentie eender wat je kan benaderen langs een gegeven URL. Denk aan de entiteiten in je functioneel domein zoals, facturen, bestellingen, dossiers, meldingen, gebruikers, etc.

### Stateless

Wanneer afnemers werken met een API, is elke request onafhankelijk van een vorige of volgende request. De API (aka server) bouwt geen "sessie" of "context" op tussen individuele requesten voor de respectievelijke afnemers (clients).

Om dit te verduidelijken beginnen we met een voorbeeld dat niet zal werken.

Stel dat we een factuur creëren door volgende `HTTP POST` Request:

``` http
POST /invoices HTTP/1.1
Content-Type: application/json
{
    "customerId": "153675ghj7",
    "invoiceDate": "..."
    "..."
}
```

Vervolgens voegen we een factuurlijn toe. 

> [!DANGER|icon:fas fa-code|label:Fout]
> ``` http
> POST /invoices/lines HTTP/1.1
> Content-Type: application/json
> {
>     "productKey": "1TER",
>     "quantity": 2
>     "..."
> }
> ```
> Dit gaat **niet** werken. 
> 
> We hebben namelijk nergens het factuurnummer meegeven dat we terug gekregen hebben van de vorige request. Stateless betekent dat de API niets gaat onthouden voor, er wordt geen context opgebouwd tussen requesten door. 

API zal niet onthouden welke factuur ik net heb gemaakt. Je moet expliciet alle data meegeven die nodig is om de volledige call af te handelen. In onderstaand voorbeeld geven we het factuurnummer mee in het path `POST /invoices/20037/lines`.

> [!TIP|icon:fas fa-code|label:Goed]
> Merk op dat er nu in de HTTP call een factuurnummer wordt meegegeven
> 
> ``` http
> POST /invoices/20037/lines HTTP/1.1
> Content-Type: application/json
> {
>     "productKey": "1TER",
>     "quantity": 2
>     "..."
> }
> ```

### Request Body, Path en methods

Even het voorbeeld herhalen:

``` http
POST /invoices/20037/lines HTTP/1.1
Content-Type: application/json
{
    "productKey": "1TER",
    "quantity": 2
    "..."
}
```

In bovenstaand voorbeeld, staat de data op verschillende plaatsen. De informatie van de factuurlijn wordt meegegeven in de body van de HTTP Post Request, terwijl het factuurnummer in het pad staat. Dit is wat wennen in het begin, Het bovenstaand voorbeeld lees je als volgt:

- **Path:** `/invoices/20037/lines` - er is een collectie van facturen met daarin één factuur met nummer 20037 en voor die factuur is er een collectie van lines (factuurlijnen)
- **Resource Method:** `POST` - en daarvoor wil ik een extra lijn toevoegen, de inhoud ervan staat in de body van de request
- **Body**: `{ "productKey": "1TER", "quantity": 2, "..."}`: met deze data maak ik de nieuwe factuurlijn aan
- **Format**: `Content-Type: application/json`: en het formaat van de data die ik meegeef is JSON
