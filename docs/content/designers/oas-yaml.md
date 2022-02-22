## Werken met YAML en OAS

Onze analyse taal is OAS ofwel Open API Specification, ook wel swagger genoemd. Met deze taal beschrijven we onze REST API in een document. De schrijfwijze die we hanteren voor dit document is YAML, hier komen we zo dadelijk op terug.

Als analist van een API is dit het belangrijkste document dat je maakt. Het kan dienen als basis voor ontwikkelaars zodat zij weten wat ze moeten maken en tegelijkertijd is het een basis voor andere stakeholders omdat je al meteen visueel kan aantonen wat er gemaakt gaat worden.

>[!TIP|icon:fas fa-forward|label:Skip dit hoofdstuk]
> Heb je de basis van OAS en YAML al achter de kiezen, ga dan meteen aan de slag met het hoofdstuk [API design](/content/designers/design).

### Tools voor de analist

Er zijn verschillende hulpmiddelen om een OAS document te maken, te valideren en je resultaten ervan te testen.

#### OAS documenten maken
[https://editor.swagger.io](https://editor.swagger.io): een eenvoudige online editor om OAS documenten mee te maken. Links zie je jouw syntax, rechts een visuele weergave van je API ontwerp.

[https://swagger.io/tools/swaggerhub](https://swagger.io/tools/swaggerhub): gaat verder dan de basis editor en laat toe om je werk eveneens te bewaren in de cloud.


#### OAS documenten valideren
[https://openapi-validator.antwerpen.be](https://openapi-validator.antwerpen.be): controleert de kwaliteit van je API ontwerp, volgens de Digipolis API regels.

#### OAS documenten testen
[https://www.getpostman.com](https://www.getpostman.com): vooral om bestaande API's te testen. Door middel van API Mocking, kan je ook je eigen ontwerp effectief testen.

>[!TIP|label:Ga meteen aan de slag]
> We duiken hieronder in de syntax van OAS. Wil je dit meteen uitproberen, open in een browser de online editor https://editor.swagger.io.

### Structuur van een OAS 3 document

Zoals eerder vermeld, wordden API's uitgeschreven volgens de [Open API Specification v3.0.3](https://swagger.io/specification). Laat ons kort inzoomen op de onderdelen van z'n file:

<a class="anchor" id="figuur-6"></a>
<p align="center">
  <img src="content/images/openapi3structure.png">
  <div align="center"><i>figuur 6 - OAS 3 Onderdelen</i></div>
</p>

- [info](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.2.md#infoObject): Bevat de basis info over de API, waaronder de naam, omschrijving en contact informatie.
- [servers](https://swagger.io/docs/specification/api-host-and-base-path/): Definities van de plaatsen waar je API staat (dev, acc, prod, etc).
- [security](https://swagger.io/docs/specification/authentication/): Beschrijft hoe je je moet authenticeren om de API te kunnen gebruiken.
- [paths](https://swagger.io/docs/specification/paths-and-operations/): Dit is het hart, hier beschrijf je aan de hand van paths (aka Routes) wat je precies met je API kan doen.
- [tags](https://swagger.io/specification/#tagObject): Je kan paths of routes klasseren volgens tags om het overzichtelijk te houden ingeval je veel routes hebt.
- [externalDocs](https://swagger.io/specification/#externalDocumentationObject): Laat toe om een extra website aan te geven waar externe documentatie staat over je API.
- [components](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.2.md#components-object): Hierin staan de herbruikbare, generieke delen van je API ontwerp zoals schema's, headers, responses, etc.

Een voorbeeld - uit de [analyse](/content/designers/analysis) - van z'n OAS document ziet er  als volgt uit:

``` yaml
openapi: 3.0.0
info:
  version: '1.0.1'
  title: 'Sales Invoice'
  description: 'Create and manage Sales Invoices..'
servers:
  - url: 'https://api-gateway/digipolis/sales-invoice/v1/...'
    description: development
paths:
  '/invoices/{number}':
    get:
      summary: Retrieve a Sales Invoice
      description: Retrieve exactly one `Sales Invoice`...
      parameters:
        - in: path
          name: number
          required: true
          schema:
            type: string
            example: 20037
      responses:
        '200':
          description: OK
      tags:
        - Invoicing
        - System
```

Je ziet een `info` luik alsook een `servers` en `paths` onderdeel in het voorbeeld hierboven, net zoals de de basis blokken van de OAS structuur. Je kan dit document op 2 manieren schrijven, ofwel in YAML ofwel in JSON. Beide kunnen, het voorbeeld van hierboven is in YAML, hetzelfde voorbeeld in JSON is hieronder.

``` json
{
  "openapi": "3.0.0",
  "info": {
    "version": "1.0.1",
    "title": "Sales Invoice",
    "description": "Create and manage Sales Invoices.."
  },
  "servers": [
    {
      "url": "https://api-gateway/digipolis/sales-invoice/v1/...",
      "description": "development"
    }
  ],
  "paths": {
    "/invoices/{number}": {
      "get": {
        "summary": "Retrieve a Sales Invoice",
        "description": "Retrieve exactly one `Sales Invoice`...",
        "parameters": [
          {
            "name": "number",
            "in": "path",
            "required": true,
            "schema": {
              "type": "string",
              "example": 20037
            }
          }
        ],
        "responses": {
          "200": {
            "description": "OK"
          }
        },
        "tags": [
          "Invoicing",
          "System"
        ]
      }
    }
  }
}
```

Beide schrijfwijzen kunnen verwerkt worden door computers, het YAML formaat heeft als voordeel dat er minder tekens worden gebruikt zoals quotes, vierkante haken en accolades. Hierdoor is het compacter en duidelijker leesbaar voor mensen. Onze voorkeur gaat dan ook uit naar het YAML formaat.

>[!WARNING]
> Je zal gemerkt hebben we in de API in het engels werken in tegenstelling tot onze analyse voorbereiding. Dit is een afspraak dat we hebben vastgelegd bij Digipolis.

### YAML 101

[YAML formaat](https://en.wikipedia.org/wiki/YAML) werkt met `key-value pairs`. Je geeft de naam van een key en vervolgens de waarde erachter.

``` yaml
  ...
  title: 'Sales Invoice'
  ...
```

Key-value pairs die behoren tot een collectie worden samen gevormd door opeenvolgende regels met dezelfde insprong  (lees: door 2 of meerdere spaties vooraan de regel toe te voegen). Zo zie je dat version, title en description behoren tot de info collection.

``` yaml
...
info:
  version: '1.0.1'
  title: 'Sales Invoice'
  description: 'Create and manage Sales Invoices..'
...
```

Tot slot zijn er lijsten, deze kan je herkennen door het `-` teken vooraan elke lijn.

``` yaml
      ...
      tags:
        - Invoicing
        - System
      ...
```

