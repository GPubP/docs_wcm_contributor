De v4 slaat op de algemene versie van het WCM (nodig voor Digipolis gateway).
De v1 slaat op de versie van de module API.

Voorlopig werkt het dus als volgt:
1. Digipolis gateway: https://api-gw-a.antwerpen.be/acpaas/wcm-proxy/v4/[pad...]   => Stuurt door naar https://wcm-gateway-app1-a.antwerpen.be/proxy/[pad...]
2. WCM gateway: https://wcm-gateway-app1-a.antwerpen.be/proxy/content/[pad...] => Stuurt door naar content module https://wcm-content-bsl-app1-a.antwerpen.be/[pad...]
3. Content BSL: `https://wcm-content-bsl-app1-a.antwerpen.be/[pad...]  (pad is bv. v1/sites/[uuid]/content?slug=[slug]&lang=[lang])

Zoals je ziet kan je momenteel nog een versie aan een tenant koppelen. Hierdoor kan elke tenant aan elke (software) versie

De wijziging aan de gateway gaat dat in de toekomst wel toelaten als de gateway een notie heeft van versies van modules bij het doorsturen.:
1. Digipolis gateway: https://api-gw-a.antwerpen.be/acpaas/wcm-proxy/v4/[pad...] => Stuurt door naar https://wcm-gateway-app1-a.antwerpen.be/proxy/[pad...]
2. WCM gateway: https://wcm-gateway-app1-a.antwerpen.be/proxy/content/v1/[pad...] => Stuurt door naar content module https://wcm-content-bsl-v1-app1-a.antwerpen.be/[pad...] (=“hardware” versioning) of https://wcm-content-bsl-app1-a.antwerpen.be/v1/[pad...] (=software versioning)
3. Content BSL: `https://wcm-content-bsl-app1-a.antwerpen.be/v1/[pad...]

Nog later gingen we meerdere contracten voorzien (~1 per module) waardoor het dan als volgt kan gaan worden:
1. Digipolis gateway: https://api-gw-a.antwerpen.be/acpaas/wcm-content/v1/[pad...]   => Stuurt door naar https://wcm-gateway-app1-a.antwerpen.be/proxy/content/v1/[pad...]
2. WCM gateway: https://wcm-gateway-app1-a.antwerpen.be/proxy/content/v1/[pad...] => Stuurt door naar content module https://wcm-content-bsl-v1-app1-a.antwerpen.be/[pad...] (=“hardware” versioning) of  https://wcm-content-bsl-app1-a.antwerpen.be/v1/[pad...] (=software versioning)
3. Content BSL: `https://wcm-content-bsl-app1-a.antwerpen.be/v1/[pad...]