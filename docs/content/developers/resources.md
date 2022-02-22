## Resources

### URI structuur
#### Benader resource volgens vastgelegde structuur <span class="rule-ref">R-US-001</span> :id=R-US-001
Resources dienen steeds te worden benaderd volgens onderstaande URI structuur
```
https://{hostname}/{namespace}/{vx}/{resource-URI}
```

#### Versioneer API URI <span class="rule-ref">R-US-002</span> :id=R-US-002
API URI's dienen steeds te worden geversioneerd (**/vx**). Aangezien wordt gekozen voor het root namespace versioneringsmodel, dient de major versie steeds te worden opgenomen in de URI en dit achter de namespace.  
De **namespace** is een enkelvoudig zelfstandig naamwoord dat het onderwerp van de API omschrijft, gezien vanuit het standpunt van de API consumer.

> Opmerking : op de API gateway bestaat de namespace uit organization/apiname

De **resource-URI** duidt op het eigenlijke resource model

Dit resulteert in onderstaand totaal voorbeeld :
```
https://api-gateway/digipolis/business-party/v1/…
```

#### Voorzie altijd de parent id(s) voor resources in een subcollection <span class="rule-ref">R-US-003</span> :id=R-US-003
Een URI voor een resource in een subcollection bevat steeds de id(s) van de parent collection(s) bij de betreffende parent collection.

> Uitzondering : event resources (groepering)

voorbeeld : 

``` http
GET https://api-gateway/digipolis/business-party/v1/business-parties/6532/contracts/42
```

#### Operaties (verbs) worden uitgevoerd op resources in de laatste collectie van het URI pad <span class="rule-ref">R-US-004</span> :id=R-US-004
Een operatie wordt uitgevoerd op één of meerdere resources in een collectie. De collectie waarop de operatie betrekking heeft, is de laatste collectie in de collectie hiërarchie; dit wordt aangegeven door in het URI pad de betreffende collectie als laatste te zetten.

In onderstaand voorbeeld haal je contract 42 op van business-party 6532.
``` http
 GET https://api-gateway/digipolis/business-party/v1/business-parties/6532/contracts/42
```

### Naming conventions
#### Gebruik zelfstandige naamwoorden voor resources, behalve voor controllers <span class="rule-ref">R-NC-001</span> :id=R-NC-001
Resources zijn in de meeste gevallen entiteiten (uitzondering : controllers, status resource)
``` http
GET /partners
```

#### Gebruik geen werkwoorden in resource namen, behalve bij controllers <span class="rule-ref">R-NC-002</span> :id=R-NC-002
Resources zijn in de meeste gevallen entiteiten (uitzondering : controllers, status resource)

> [!DANGER|icon:fas fa-code|label:Fout]
> ``` http
> GET /getpartners
> ```
> gebruik je dus NIET

#### Definieer resources in het meervoud <span class="rule-ref">R-NC-003</span> :id=R-NC-003
Resources worden steeds in het meervoud gedefinieerd (uitzondering : controllers, status resource)
``` http
GET /partners
```

#### Gebruik steeds lowercase voor URI en query parameters <span class="rule-ref">R-NC-004</span> :id=R-NC-004
Dit vermijdt technologie afhankelijke problemen met casing.
``` http
GET /partners?page=10&pagesize=20
```

#### Gebruik geen underscores "\_" of dots "." in de URI <span class="rule-ref">R-NC-005</span> :id=R-NC-005
``` http
GET /business-parties?page=10&pagesize=20
```

#### Gebruik hyphenation om woorden van elkaar te scheiden <span class="rule-ref">R-NC-006</span> :id=R-NC-006
Dit verhoogt de leesbaarheid van de URI.
``` http
GET /business-parties?page=10&pagesize=20
```

#### Gebruik geen trailing slash in de URI <span class="rule-ref">R-NC-007</span> :id=R-NC-007
Een trailing slash heeft geen toegevoegde waarde en verlaagt bovendien de leesbaarheid van de URI.

> [!DANGER|icon:fas fa-code|label:Fout]
> ``` http
> GET /partners/
> ```
> gebruik je dus NIET

#### Gebruik geen fragments in de URI <span class="rule-ref">R-NC-008</span> :id=R-NC-008
Fragments (\#) worden gebruikt om te navigeren binnen een web context pagina, maar mogen niet gebruikt worden in een API URI.

> [!DANGER|icon:fas fa-code|label:Fout]
> ``` http
> GET /partners#name/
> ```
> gebruik je dus NIET

### Media types en content negotiation
#### Gebruik steeds media types en content negotiation <span class="rule-ref">R-MC-001</span> :id=R-MC-001
Media types en content negotiation dient **steeds** te gebeuren voor elke API call. Media types worden steeds gecommuniceerd via de HTTP Content-Type en Accept headers.

#### Gebruik nooit file extenties in de URI om media types te communiceren <span class="rule-ref">R-MC-002</span> :id=R-MC-002

> [!DANGER|icon:fas fa-code|label:Fout]
> ``` http
> GET /partners.json
> of 
> GET /partners.xml
> ```
> gebruik je dus NIET

#### Gebruik standaard het application/json media type <span class="rule-ref">R-MC-003</span> :id=R-MC-003
Dit maakt dat de resource representatie standaard JSON is.

#### Definieer optioneel een locale <span class="rule-ref">R-MC-004</span> :id=R-MC-004
Definitie van een locale is optioneel. Dit zorgt er voor dat locatie specifieke formaten zoals datums of geldeenheden op een correcte manier worden voorgesteld.

Communicatie van de locale gebeurt steeds door middel van de **Accept-Language** en **Content-Language** headers.

De keuze van locale is API specifiek.

**Om onze services zo breed mogelijk bruikbaar te maken, is gekozen om alle API's in het Engels te maken**.

#### Gebruik specifieke HTTP headers voor media-types en content negotiation <span class="rule-ref">R-MC-005</span> :id=R-MC-005
**Content-Type**: definiëert het formaat van het request & response bericht. Communicatie is zowel van consumer naar provider als van provider naar consumer.  
**Content-language**: definiëert de locale van het request & response bericht. Communicatie is zowel van consumer naar provider als van provider naar consumer.  
**Accept**: definieert de formaten van het response bericht dewelke de API consumer begrijpt en die door de API provider kunnen worden teruggestuurd. Communicatie is steeds van consumer naar provider.  
**Accept-language**: definieert de locale van het response bericht dewelke de API consumer begrijpt en dat door de API provider kan worden teruggestuurd. Communicatie is steeds van consumer naar provider.  

**application/json** is het geprefereerde media type formaat.
