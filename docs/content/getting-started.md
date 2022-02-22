# Getting started

Het API system van Digipolis is een verzameling van zowel informatie als hulpmiddelen die je kan gebruiken tijdens het ontwerpen en bouwen van API's.

Ben je een `Developer`, kijk dan zeker naar de [API requirements](/content/developers/intro.md), de [API Validator](/content/tools/validate.md) en de [API Generator](/content/tools/generators.md)

!> De huidige API engine (Store en Publisher) ondersteunt enkel Swagger 2.0. Indien je jouw API hierop publiceert, moet je dus het Swagger 2.0 formaat volgen.

Ben je een `Designer`, of wil je leren hoe je een API moet ontwerpen, lees dan de [design hoofdstukken](/content/designers/intro.md). 

Ben je helemaal nieuw, start dan eerst [bij de basics](/content/common/api.md).

## Waarom hebben we dit gemaakt?

Het is belangrijk bij het ontwikkelen van APIs dat het ontwikkelteam consistent is in :
-   naamgevingen (resources altijd meervoud (uitzondering : controllers, status resource), lowercase...)
-   formatteringen van datums, geolocaties,...
-   het implementeren van paginatie/size limiet om de hoeveelheid data die in 1 keer opgevraagd wordt te beperken
-   het optioneel maken van velden die niet voor alle consumers even nuttig zijn
-   het antwoorden met een gestandaardiseerd error contract bij elke API met de bijbehorende juiste HTTP status codes
-   een correct gebruik van verbs, ...

Dit zorgt voor een verhoogde ease-of-use en kan de consuming developers heel wat frustraties en opzoekwerk besparen.

Voor interne API's betekent dit dat de consumers een kortere ontwikkeltijd nodig hebben om met jou API te integreren.

Voor publieke API's is dit des te meer belangrijk omdat je hierdoorzorgt voor een betere "developer experience" en je een hogere
adoptiegraad kan faciliteren.


## Contributing & Feedback

Heb je nog een vraag of een idee om iets te verbeteren/bij te dragen? Kijk even naar de [contributing guide](/content/contributing.md)

## Changelog

Bekijk de changelog voor de [API Requirements](/content/developers/change-log.md)

Bekijk de changelog voor de [API Design Guide](/content/designers/change-log.md)