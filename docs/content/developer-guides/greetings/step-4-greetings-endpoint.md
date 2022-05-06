# Hoofdstuk 4: Greetings endpoint voorzien

We hebben nu een [eigen BSL service draaien binnen de WCM context](/content/developer-guides/greetings/step-3-wcm-module-setup.md).

Binnen dit hoofdstuk gaan we een greetings endpoint beschikbaar stellen om een begroeting op te vragen.\
We voorzien hierbij de nodige rechten en rechten checks zodat enkel gebruikers met een `greetings-read` recht toegang krijgen tot de endpoint.

## Stap 1: NestJS module opzetten
Als eerste stap zullen we een Greetings NestJS module opzetten waarin onze greetings endpoint zal bestaan.\
We starten alvast met de volgende file aan te maken `/server/src/modules/greetings/greetings.module.ts`:

```ts
// server/src/modules/greetings/greetings.module.ts
import { Module } from '@nestjs/common';

@Module({
  imports: [],
  controllers: [],
  providers: [],
})
export class GreetingsModule {}
```

Om deze module van functionaliteit te voorzien moeten we een controller aanmaken waarin we endpoints kunnen maken.\
Maak hiervoor de volgende file aan `/server/src/modules/greetings/v1/controllers/greetings.controller.ts`:

```ts
// server/src/modules/greetings/v1/controllers/greetings.controller.ts
import { Controller } from '@nestjs/common';
import {
	ApiBadRequestResponse,
	ApiInternalServerErrorResponse,
	ApiNotFoundResponse, ApiTags,
	ApiUnauthorizedResponse,
} from '@nestjs/swagger';

import { GenericErrorResponse } from '~shared/classes/Error';

@Controller(`${GreetingsController.version}/greetings`)
@ApiTags('Greetings')
@ApiUnauthorizedResponse({ type: GenericErrorResponse, description: 'Unauthorized' })
@ApiBadRequestResponse({ type: GenericErrorResponse, description: 'Bad Request' })
@ApiInternalServerErrorResponse({ type: GenericErrorResponse, description: 'Internal Server Error' })
@ApiNotFoundResponse({ type: GenericErrorResponse, description: 'Not Found' })
export class GreetingsController {
    static version = 'v1';
}
```

Nu we deze greetings controller hebben, kunnen we deze kenbaar maken binnen onze Greetings module als volgt:

```ts
// server/src/modules/greetings/greetings.module.ts
import { Module } from '@nestjs/common';

import { GreetingsController } from './controllers/greetings.controller';

@Module({
  imports: [],
  controllers: [GreetingsController],
  providers: [],
})
export class GreetingsModule {}
```

Onze greetings module heeft nu een controller maar deze module wordt zelf nog niet ingeladen.\
We moeten dus de file `/server/src/core/core.module.ts` aanpassen zodat de greetings module deel zal uitmaken van de BSL service bij het opstarten:

```ts
// server/src/core/core.module.ts
import { Module } from '@nestjs/common';

import { GreetingsModule } from '~modules/greetings/v1/greetings.module';

import { StatusController } from './controllers/status.controller';

@Module({
	imports: [GreetingsModule],
	controllers: [StatusController],
})
export class AppModule {}
```
## Stap 2: Module beveiligen

Onze module is nu opgezet maar deze gaat by default alle calls toelaten, ook calls die niet via de wcm-gateway verlopen.\

> [!danger] **Deze stap is heel belangrijk.**
>
> Voor de veiligheid van de module en het WCM in het algemeen. Deze middleware functies, aangeboden door de `@wcm/config-helper`, zorgen ervoor dat enkel calls aanvaardt worden die eerst langs de WCM Gateway gepasseerd zijn of de juiste tenant key bevatten.
>
> Dit is belangrijk omdat de WCM gateway een aantal cruciale checks uitvoert voor de request doorgestuurd wordt naar de BSL module:
>  - Gaat op basis van de route, contract, tenant en credentials na of deze route wel toegankelijk is
>  - Vervangt een apikey (of application identifier) door een tenant context in de vorm van een JWT token

**Dit mag dus niet!** Daarom voegen we de volgende middleware toe aan onze module:

```ts
// server/src/modules/greetings/greetings.module.ts
import { MiddlewareConsumer, Module } from '@nestjs/common';

import { tenantConfig } from '~shared/helpers/tenant';

import { GreetingsController } from './controllers/greetings.controller';

@Module({
  ...
})
export class GreetingsModule {
	configure(consumer: MiddlewareConsumer) {
		consumer
			.apply(
				tenantConfig.verifyJwt(),
				tenantConfig.apiKeyGuard
			)
			.forRoutes(GreetingsController);
	}
}
```

## Stap 3: Route aanmaken
Nu we alles opgezet hebben, kunnen we een Greetings endpoint als volgt aanmaken in onze controller:

```ts
// server/src/modules/greetings/v1/controllers/greetings.controller.ts
...
@Controller(`${GreetingsController.version}/greetings`)
...
export class GreetingsController {
	static version = 'v1';

	@Get()
	public async getGreeting(): Promise<any> {
		return {
			message: 'Welkom',
		};
	}
}
```

## Stap 4: Endpoint documenteren

We kunnen deze call even testen door de volgende call uit te voeren:
```bash
curl --location --request GET 'http://localhost:7200/proxy/admin/greetings/v1/greetings' \
    --header 'apikey: 000-000'
```

We krijgen nu een `404 Not found` terug. Dit komt omdat de gateway nog geen weet heeft van deze call.\
De gateway laat enkel calls door die terug te vinden zijn in de swagger documentatie van onze BSL service.

De swagger documentatie van de BSL service vind je hier:
- http://localhost:60101/docs/admin/
- http://localhost:60101/docs/internal/
- http://localhost:60101/docs/public/

We willen dat onze greetings call terugkomt onder `http://localhost:60101/docs/admin/` zodat we deze call kunnen aanpreken via de `http://localhost:7200/proxy/admin/` prefix.

Om dit te realiseren, moeten we een `@ApiAccess()` decorator toevoegen aan onze call met de markering dat deze call beschikbaar gesteld mag worden onder de admin prefix:

```ts
// server/src/modules/greetings/v1/controllers/greetings.controller.ts
...
@Controller(`${GreetingsController.version}/greetings`)
...
export class GreetingsController {
	static version = 'v1';

	@Get()
    @ApiAccess(apiAccessMap.ADMIN)
	public async getGreeting(): Promise<any> {
		return {
			message: 'Welkom',
		};
	}
}
```

Als we nu de volgend request uitvoeren krijgen we ons bericht correct terug:

```bash
curl --location --request GET 'http://localhost:7200/proxy/admin/greetings/v1/greetings' \
    --header 'apikey: 000-000'
```
```json
{
    "message": "Welkom"
}
```

Als we surfen naar `http://localhost:60101/docs/admin` en `http://localhost:7200/docs/tenant-test-1/admin/#/Greetings/GreetingsController_getGreeting` zien we het resultaat van onze documentatie.

![Documentatie van greetings module](../../../assets/greetings-module-bsl-docs-1.png ':size=900')

We zien al snel dat onze documentatie nog niet af is.
1. De titel is verkeerd en bevat nog de naam `Boilerplate`.
2. De documentatie van onze call is niet echt goed. Er is geen output schema bijvoorbeeld.

De titel aanpassen kan onder de `mountSwagger` methode in de `/server/src/app.ts` file:

```ts
// server/src/app.ts
...
private async mountSwagger(): Promise < void > {
    const options = new DocumentBuilder()
        .setTitle('Greetings BSL')
        .setDescription('Greetings BSL service dat je een gepaste begroeting kan geven')
        .setVersion(version)
        .build();
    const document = SwaggerModule.createDocument(this.app, options);
    hostSwagger(this.app, document);
}
```

De documentatie kunnen we verbeteren door de volgende decorators toe te voegen aan onze controller `getGreetings` methode:

```ts
// server/src/modules/greetings/v1/controllers/greetings.controller.ts
...
    @Get()
    @ApiOperation({ summary: 'Get a greeting', description: 'Get a greeting from a tenant' })
    @ApiOkResponse({ type: Object })
    @ApiAccess(apiAccessMap.ADMIN)
    public async getGreeting(): Promise<any> {
        return {
            message: 'Welkom',
        };
    }  
```

Dat is al iets beter maar nog niet helemaal wat we willen. De response schema is nu een leeg object maar dat klopt niet.\
Om dit op te lossen voorzien we een klasse dat onze response beschrijft en voegen we het toe als type aan onze `@apiOkResponse`.

```ts
// server/src/modules/greetings/v1/classes/Greeting.ts
import { ApiProperty } from '@nestjs/swagger';

export class Greeting {
	@ApiProperty()
	message: string;
}
```
```ts
// server/src/modules/greetings/v1/controllers/greetings.controller.ts
...
	@Get()
	@ApiOperation({ summary: 'Get a greeting', description: 'Get a greeting from a tenant' })
	@ApiOkResponse({ type: Greeting })
	@ApiAccess(apiAccessMap.ADMIN)
	public async getGreeting(): Promise<Greeting> {
		return {
			message: 'Welkom',
		};
	}
```

Nu ziet onze documentatie er al een pak beter uit:

![Documentatie van greetings module zoals het moet](../../../assets/greetings-module-bsl-docs-2.png ':size=900')

> [!note]
> Merk op dat je de greetings endpoint niet rechtreeks kan opvragen.
>
> ```bash
> curl --location --request GET 'http://localhost:60101/v1/greetings
> ```
> ```json
> {
>    "type": "/errors/v1/Unauthorized",
>     "title": "You are not authorized to use this endpoint",
>     "status": 401,
>     "identifier": "f03b4579-87fb-4204-8884-8adf7aed8513",
>     "code": "Unauthorized"
> }
> ```
>
> Dit komt door onze `verifyJwt` en `apiKeyGuard` middleware dat we voorheen toegevoegd hebben aan onze greetings module.
> 
> Merk ook op dat de status call wel nog steeds rechtstreeks toegankelijk is:
> ```bash
> curl --location --request GET 'http://localhost:60101/status'
> ````
> Dit is ok aangezien een status call ook moet kunnen werken als de gateway onbereikbaar is.

## Stap 5: Rechten aanmaken

Onze call is nu correct aangemaakt en enkel toegankelijk via de wcm-gateway maar onze call wordt nog niet beschermd achter een security-right.\
De call is dus toegankelijk voor iedereen dat toegang heeft tot de tenant en niet voor enkele gebruikers die tot een bepaalde rol behoren.

In dit geval zou dit ok zijn maar ter illustratie gaan we toch onze call beschermen achter een recht.

Als eerste stap moeten we de rechten kenbaar maken bij BraaS (het rollen & rechten systeem van het WCM).
Dit kunnen we doen door de `@wcm/braas-bsl-helper` helper te gebruiken als volgt in de Greetings module:

```ts
// server/src/modules/greetings/greetings.module.ts
...
@Module({
  ...
})
export class GreetingsModule {
	constructor() {
		if (!tenantConfig.getModuleContext()) {
			tenantConfig.on('ready', () => this.registerPermissions());
		} else {
			this.registerPermissions();
		}
	}

	private async registerPermissions() {
		// tslint:disable-next-line: no-console
		console.log('REGISTERING GREETINGS PERMISSION ...');

		const tenantResult = await registerTenantPermissions({
			name: 'greetings',
			label: 'Greetings',
		}, [{
			key: 'greetings_read',
			name: 'Greetings read',
			description: 'Allow reading a greeting',
			type: 'module',
			applyToDefaultRoles: ['Contentbeheerder'],
			derivedBackendRights: [],
		}]).catch(() => null);

		// tslint:disable-next-line: no-console
		console.log('GREETINGS PERMISSIONS REGISTERED WITH RESULT:', tenantResult);
	}
    ...
```

Na het herstarten van de server zal je de volgende logs zien doorkomen:
```bash
wcm-boilerplate-bsl | REGISTERING GREETINGS PERMISSION ...
wcm-boilerplate-bsl | GREETINGS PERMISSIONS REGISTERED WITH RESULT: {
wcm-boilerplate-bsl |   created: { fulfilled: [ [Object] ], rejected: [] },
wcm-boilerplate-bsl |   updated: { fulfilled: [], rejected: [] },
wcm-boilerplate-bsl |   removed: { fulfilled: [], rejected: [] }
wcm-boilerplate-bsl | }
```

Je ziet onder `created.fulfilled` dat een recht is aangemaakt geweest.

## Stap 6: Route afschermen achter een recht

Nu we een recht geregistreerd hebben bij BraaS, kunnen we onze route afschermen achter dit recht.

Als eerste moeten we een guard toevoegen aan onze greetings module.\
De `SecurityRightGuard` zal routes afschermen die aangeven dat er rechten gechecked moeten worden (zie verder).

```ts
// server/src/modules/greetings/greetings.module.ts
import { APP_GUARD } from '@nestjs/core';

import { SecurityRightGuard } from '~shared/decorators/SecurityRightGuard';
...
@Module({
  imports: [],
  controllers: [GreetingsController],
  providers: [{
		provide: APP_GUARD,
		useClass: SecurityRightGuard,
	}],
})
export class GreetingsModule {
    ...
}
```

Nu de guard ingesteld is, kunnen we aan onze greetings endpoint zo instellen dat de gebruiker, die deze call aanspreekt, een `greetings_read` recht nodig heeft binnen de tenant.
Dit kunnen we doen door de `@SecurityRights(...rights)` decorator toe te voegen aan onze route.

```ts
// server/src/modules/greetings/v1/controllers/greetings.controller.ts
...
    @Get()
	@ApiOperation({ summary: 'Get a greeting', description: 'Get a greeting from a tenant' })
	@ApiOkResponse({ type: Greeting })
	@ApiAccess(apiAccessMap.ADMIN)
	@SecurityRights('greetings_read')
	public async getGreeting(): Promise<Greeting> {
		return {
			message: 'Welkom',
		};
	}
```

Als we nu onze greetings endpoint aanspreken krijgen wen een `403 Forbidden` terug:

```bash
curl --location --request GET 'http://localhost:7200/proxy/admin/greetings/v1/greetings' \
    --header 'apikey: 000-000'
```
```json
{
    "response": {
        "statusCode": 403,
        "message": "Forbidden resource",
        "error": "Forbidden"
    },
    "status": 403,
    "message": "Forbidden resource"
}
```

Dit is logisch want we hebben aan onze service niet laten weten wie we zijn.\
Hierdoor weet de server niet of we al dan niet de juiste rechten hebben.\
We moeten dus een bearer token meegeven van onze gebruiker (die tenant- of content beheerder is) in de `Authorization` header:

```bash
curl --location --request GET 'http://localhost:7200/proxy/admin/greetings/v1/greetings' \
    --header 'apikey: 000-000' \
    --header 'Authorization: Bearer [user-bearer-token]'
```

Het verkijgen van deze token kan via de m-profiel API.\
Als alternatief gaan we dit testen door een test te schrijven en later onze frontend module te laten connecteren met deze service.

## Stap 7: Test schrijven

Om ons nieuwe endpoint te testen maken we een nieuwe spec file `/server/test/integration/greetings/greetings.spec.ts`:

```ts
// server/test/integration/greetings/greetings.spec.ts
import supertest from 'supertest';

import { App } from '~app';
import config from '~config';

let api: supertest.SuperTest<supertest.Test>;

describe('[INTEGRATION - LANGUAGES] Languages endpoint', () => {
	let app: App;

	beforeAll(async() => {
		app = new App();
		await app.bootstrapCore();
		api = supertest(`http://localhost:${config.server.port}`);
		await wait(500);
	});

	afterAll(() => {
		app.server.close();
	});

	it('should return "Welkom"', async () => {
		const result = await api.get('/v1/greetings').expect(200);

		expect(result).toEqual({
			message: 'Welkom',
		});
	});
});
```

We kunnen onze test runnen door het volgende commando te runnen in de `/server` folder:
```bash
cd ./server
```
```bash
npm run test
```

Onze test faalt met een de melding dat we een `401` terugkrijgen i.p.v. een `200`. 
Dit komt omdat we geen JWT gateway token en tenant key meegeven aan de request.

```ts
// server/test/integration/greetings/greetings.spec.ts
...
it('should return "Welkom"', async () => {
    const jwtToken = await getMockJwtGatewaytoken();
    const result = await api.get('/v1/greetings')
		.set('apikey', moduleConfig.appsAccess[0].apikey)
        .set('Authorization', jwtToken)
        .expect(200);

    expect(result).toEqual({
        message: 'Welkom',
    });
});
```

We krijgen hierna nog steeds de melding dat we een `401` terugkrijgen i.p.v. een `200`. Dit komt omdat onze call bij BraaS BSL gaat nagaan of de gebruiker wel de `greetings_read` recht heeft.

Aangezien we een in een test context zitten, moeten we dus nog een mock voorzien zodat de security right guard een fictieve response krijgt van de BraaS BSL.
```ts
// server/test/integration/greetings/greetings.spec.ts
...
beforeEach(() => {
    nockTenantRights(['greetings_read']);
});

it('should return "Welkom"', async () => {
    const jwtToken = await getMockJwtGatewaytoken();
    const result = await api.get('/v1/greetings')
        .set('apikey', moduleConfig.appsAccess[0].apikey)
        .set('Authorization', jwtToken)
        .expect(200);

    expect(result).toBeDefined();
    expect(result.body).toEqual({
        message: 'Welkom',
    });
});
```

Nu slaagt onze test en zijn we klaar met de greetings endpoint!