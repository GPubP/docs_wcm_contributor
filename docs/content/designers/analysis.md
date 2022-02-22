## Analyse voorbereiding

Hoe begin je nu aan een API? Wel, eerst en vooral moet je het functioneel domein omzetten in een ontwerp.

>[!TIP|icon:fas fa-forward|label:Skip dit hoofdstuk]
> Ben je al gewoon om functionele analyses te maken, sla dan dit hoofdstuk over en ga meteen aan de slag met het [API design](/content/designers/design) of lees eerst een intro in het werken met [OAS en YAML](/content/designers/oas-yaml).

De analyse gaan we doen aan de hand van een voorbeeld. We maken hierbij gebruik van [UML Class diagrams](https://en.wikipedia.org/wiki/Class_diagram) als notatievorm. In se maakt het niet uit wat je hier voor gebruikt, al waren het bierkaartjes ;).

### Zoek de entiteiten

In veel analyse technieken zoals [Database Normalization](https://en.wikipedia.org/wiki/Database_normalization) of [Domain Driven Design](https://en.wikipedia.org/wiki/Domain-driven_design) begin je met het zoeken van `entiteiten` vanuit een functioneel domein.


>[!TIP|icon:fas fa-book-open|label:Je krijgt als analyst de volgende vraag]
> *Ik wil een systeem om `verkoopfacturen` mee te kunnen maken. Een factuur bevat een logo, een uniek nummer, is steeds voor één specifieke `klant` op een gegeven factuurdatum en bevat één of meerdere `factuurlijnen`. Als ik 2 verschillende `producten` verkoop aan de klant, staan deze elk apart vermeld in 2 verschillende factuurlijnen. Elke factuurlijn toont de product code, een omschrijving, het aantal stuks dat er van verkocht is, de eenheidsprijs van het product en de totaalprijs (aantal x productprijs). Op het einde van de factuur staat het bedrag exclusief BTW, de BTW toelage en de totaalprijs inclusief BTW.*

In bovenstaande tekst vinden we volgende entiteiten terug:

>[!NOTE|icon:fas fa-info-circle|label:Entiteiten]
>``` plantuml
>@startuml
>class Factuur 
>class Factuurlijn
>class Klant
>class Product
>@enduml
>```

### Relaties tussen entiteiten

Neem de zin *"Een factuur bevat een logo, een uniek nummer, is steeds voor één specifieke `klant` op een gegeven factuurdatum en bevat één of meerdere `factuurlijnen`. Als ik 2 verschillende producten verkoop aan de klant, staan deze elk apart vermeld in 2 verschillende factuurlijnen."*.

Hieruit lezen we dat een factuur een klant, en factuurlijnen heeft. Er is dus een relatie tussen beiden. Daarnaast zien we ook dat er een relatie is tussen een factuurlijn en een product.

>[!NOTE|icon:fas fa-info-circle|label:Relaties]
>``` plantuml
>@startuml
>class Factuur 
>class Factuurlijn
>class Klant
>class Product
>Factuur "1" -down- "1..*" Factuurlijn
>Factuur "0..*" -down- "1" Klant
>Factuurlijn "0..*" -down- "1" Product
>@enduml
>```

Bovenstaand schema lezen we als volgt:

- Een factuur is voor één klant, en die klant kan nul of meerdere facturen hebben
- Een factuur heeft minstens één of meerdere factuurlijnen, en een factuurlijn behoort steeds tot één factuur
- Een factuurlijn is voor één product, en een product kan voorkomen op geen of meerdere factuurlijnen.

Wanneer we hierboven spreken over "nul", "één" of "meerdere" noemen we dat ook wel eens [cardinaliteit](https://en.wikipedia.org/wiki/Cardinality_(data_modeling))

### Aggregatie en compositie

> [!WARNING]
> We maken hier natuurlijk een aantal assumpties. In de praktijk zal je die uiteraard afstemmen met de personen voor wie je de toepassing creëert.


Als we het voorgaande nog verder modelleren, kunnen we volgende stellen:

- Een klant kan bestaan zonder dat deze ooit een factuur heeft
- Een product kan bestaan in het systeem zonder dat het ooit op een factuur is voorgekomen
- Sterker nog, het heeft zelfs geen enkel bestaansrecht op zichzelf.

Deze zaken drukken we uit via aggregatie en compositie. Maw,

- **aggregatie:** de entiteit in een relatie heeft een bestaansrecht ook buiten haar relatie
- **compositie:** de entiteit in een relatie heeft geen bestaansrecht buiten haar relatie

Dit wordt weergegeven door open of gesloten ruiten in het diagram

>[!NOTE|icon:fas fa-info-circle|label:Aggregatie en compositie]
>``` plantuml
>@startuml
>class Factuur 
>class Factuurlijn
>class Klant
>class Product
>Factuur "1" *-down- "1..*" Factuurlijn
>Factuur "0..*" o-down- "1" Klant
>Factuurlijn "0..*" o-down- "1" Product
>@enduml
>```

Later in het ontwerp van de API gaan we zien waarom dit verschil nu net belangrijk is.

### Zoek de eigenschappen van een entiteit

De zin *"Een factuur bevat een `logo`, een `uniek nummer`, is steeds voor één specifieke `klant` op een gegeven `factuurdatum` en bevat één of meerdere factuurlijnen"* beschrijft ook de eigenschappen van een **factuur**.

- een logo
- (uniek) factuurnummer
- een factuurdatum

Vanuit de oorspronkelijke vraag komen we dan bij het volgende diagram uit:

>[!NOTE|icon:fas fa-info-circle|label:Eigenschappen]
>``` plantuml
>@startuml
>class Factuur {
>  nummer: string
>  datum: date
>  logo: url
>  klant: Klant
>  factuurlijnen: Factuurlijn[]
>}
>class Factuurlijn{
>  aantal: integer
>  product: Product
>}
>class Klant
>class Product{
>  omschrijving: string
>  eenheidsprijs: currency
>}
>Factuur "1" *-down- "1..*" Factuurlijn
>Factuur "0..*" o-down- "1" Klant
>Factuurlijn "0..*" o-down- "1" Product
>@enduml
>```

In essentie is dit het soort werk dat we als analist doen. In volgende hoofdstukken gaan we dit analyse ontwerp in detail omzetten in een elegante API. Maar daarvoor moeten we ons eerst in de juiste mindset krijgen, vandaar de Design Principes.

