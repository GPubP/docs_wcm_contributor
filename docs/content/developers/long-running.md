## Langdurende operaties
### Asynchrone verwerking
#### Gebruik Asynchrone verwerking voor langdurende operaties <span class="rule-ref">R-AV-001</span> :id=R-AV-001
API operaties dienen steeds zo kort mogelijk worden gehouden. Indien bepaalde operaties toch langer zouden duren, zoals bijvoorbeeld update operaties, dan kunnen deze asynchroon worden afgehandeld indien functioneel mogelijk.

#### Gebruik minimaal HTTP status code `202 Accepted` <span class="rule-ref">R-AV-002</span> :id=R-AV-002
In REST wordt HTTP status code 202 gebruikt om een asynchrone verwerking aan te geven.

#### Implementeer eventueel een polling mechanisme voor statusopvolging <span class="rule-ref">R-AV-003</span> :id=R-AV-003
Meestal zal bij asynchrone verwerking een API de consumer toelaten om de status van de verwerking op te volgen.
Hierbij wordt gebruik gemaakt van de HTTP `Location` header en HTTP status codes om de toestand weer te geven.

Vanuit technisch perspectief kan volgende flow gevolgd worden :

1.  `De API consumer voert een request uit: POST /{resources}`
2.  `De API consumer ontvangt volgend antwoord: 202 Accepted, Location: /tasks/{id} Hierbij bevat de Location header de URI van een tijdelijke resource.`
3.  `De API consumer polt op de url /tasks/{id}. Indien de taak nog niet uitgevoerd is, is de response 200 OK met een body die de status van de taak beschrijft.`
4.  `Wanneer de taak uitgevoerd is, krijg je een response 303 See Other, Location: /{resources}/{resource-id}. De Location header bevat in dit geval de URI van de definitieve resource.  Wanneer er een fout is gebeurd bij het uitvoeren van de taak, krijg je een error response met de details van het probleem en de toepasselijke statuscode.`
5.  `De API consumer kan een request uitvoeren op de gegeven url : GET /resources/{resource-id} om de resource representatie op te halen.`
