# Hoofdstuk 5: Greetings endpoint aanspreken

We hebben nu een BSL die een `/v1/greetings` endpoint beschikbaar stelt.
Als laatste stap willen we deze endpoint aanspreken vanuit onze greetings frontend module (zie [hoofdstuk 1](/content/developer-guides/step-1-redactie-module-setup.md) en [hoofdstuk 2](/content/developer-guides/step-2-greetings-page.md)).

## Stap 1: Redactie naar lokale setup verwijzen.

Indien je bij het bouwen van de greetings frontend module gebruik maakte van de dev backend, moet je de Redactie app aanpassen zodat deze niet met DEV verbindt maar met onze lokale setup verbindt (zie ook: [Omgevingen](http://localhost:4000/#/content/setup/redactie/setup?id=omgevingen)).

In de redactie app moeten we de docker-compose.yml file als volgt voorzien:
```yaml
# ...
services:
  # ...
  server:
    env_file: local.env
    # ...
```

Dit gaat configuratie inladen zodat de Redactie zal verwijzen naar:
- http://localhost:3999 i.p.v. https://api-gw-o.antwerpen.be/district01/wcm-admin/v4 (= WCM Admin)
- https://localhost:7200 i.p.v. https://api-gw-o.antwerpen.be/district01/wcm-content-manager/v4 (= WCM gateway)
- enz

Na deze instellingen gedaan te hebben moet je de redactie app even herstarten.
```bash
docker-compose down && docker-compose up
```

## Stap 2: Greetings service

Om onze greetings BSL aan te spreken, maken we een Greetings api service dat de call doet naar ons nieuw `/v1/greetings` endpoint.

- `/public/lib/services/greetings/greetings.service.types.ts`

    Voorzie een type voor de response die we zullen terug krijgen:
    ```ts
    // public/lib/services/greetings/greetings.service.types.ts
    export interface GreetingsResponse {
        message: string;
    }
    ```
- `/public/lib/services/greetings/greetings.service.const.ts`

    De prefix van onze url volgt de volgende pattern `[route-prefix van BSL module]/[path vanaf root van de BSL module]`.\
    Hierdoor is het voor de greetings module als volgt:
    ```ts
    // public/lib/services/greetings/greetings.service.const.ts
    export const GREETINGS_PREFIX_URL = 'greetings/v1/greetings';
    ```
- `/public/lib/services/greetings/greetings.service.ts`

    We voorzien een service dat de call doet naar de Greetings BSL om een greeting te voorzien.
    ```ts
    // public/lib/services/greetings/greetings.service.ts
    import { api } from '../api';

    import { GREETINGS_PREFIX_URL } from './greetings.service.const';
    import { GreetingsResponse } from './greetings.service.types';

    export class GreetingsApiService {
        public async getGreeting(): Promise<GreetingsResponse> {
            return api.get(GREETINGS_PREFIX_URL).json();
        }
    }

    export const greetingsApiService = new GreetingsApiService();
    ```
- `/public/lib/services/greetings/index.ts`

    Als laatste exporteren we de service en zijn typings naar buiten de greetings service folder.
    ```ts
    // public/lib/services/greetings/index.ts
    export * from './greetings.service.types';
    export * from './greetings.service';
    ```

## Stap 3: Store

We hebben nu een service om de Greetings BSL aan te spreken.\
Nu moeten we nog een store en facade opzetten zodat onze pagina aan de data kan die deze service op zal halen.

We maken gebruik van Akita voor onze state management. We moeten deze dus eerst installeren als dev dependency.

```bash
npm i @datorama/akita -D
```

> [!note]
> We installeren Akita als dev dependency omdat we deze niet mee hoeven te packagen binnen onze bundel.\
> Binnen de Redactie app is Akita reeds aanwezig in zijn core bundel. De App zorgt ervoor dat deze instantie van Akita toegankelijk is binnen onze module.

Eens Akita geïnstalleerd is, kunnen we onze store aanmaken door de volgende bestanden te voorzien:

- `/public/lib/store/greetings/greetings.model.ts`

    Voor we de store opzetten voorzien we een model waarop de store zal gebasseerd worden. Voorlopig is dit model nog heel basic en hergebruiken we de `GreetingsResponse` interface hiervoor.

    De `BaseMultiEntityState<Model, IDType>` voorziet een interface dat gebruikt kan worden in de `BaseMultiEntityStore<StateType>` (zie volgend punt).
    ```ts
    // public/lib/store/greetings/greetings.model.ts
    import { BaseMultiEntityState } from '@redactie/utils';

    import { GreetingsResponse } from '../../services/greetings';

    export type GreetingModel = GreetingsResponse;
    export type GreetingsState = BaseMultiEntityState<GreetingModel, string>;
    ```

- `/public/lib/store/greetings/greetings.store.ts`

    Nu we een model (en state) hebben, kunnen we een akita store opzetten.\
    We extenden van de `BaseMultiEntityStore` die reeds heel wat boilerplate voor ons voorziet.
    ```ts
    // public/lib/store/greetings/greetings.store.ts
    import { StoreConfig } from '@datorama/akita';
    import { BaseMultiEntityStore } from '@redactie/utils';

    import { GreetingsState } from './greetings.model';

    @StoreConfig({ name: 'greetings', idKey: 'id' })
    export class GreetingsStore extends BaseMultiEntityStore<GreetingsState> {
        constructor(initialState: Partial<GreetingsState>) {
            super(initialState);
        }
    }

    export const greetingsStore = new GreetingsStore({});
    ```

- `/public/lib/store/greetings/greetings.query.ts`

    Om de store te kunnen bevragen voorzien we een aparte query service die extend van `BaseMultiEntityQuery<StateType>`.\
    De `BaseMultiEntityQuery` klasse voorziet opnieuw heel wat boilerplate voor ons.
    ```ts
    // public/lib/store/greetings/greetings.query.ts
    import { BaseMultiEntityQuery } from '@redactie/utils';

    import { GreetingsState } from './greetings.model';
    import { greetingsStore } from './greetings.store';

    export class GreetingsQuery extends BaseMultiEntityQuery<GreetingsState> {}

    export const greetingsQuery = new GreetingsQuery(greetingsStore);
    ```

- `/public/lib/store/greetings/greetings.facade.ts`

    Nu de store opgezet is, voorzien we een facade die de store, query & API service zal aanspreken.\
    Het is de bedoeling dat onze toepassing altijd de facade zal aanspreken en nooit rechtstreeks een van deze achterliggende services.
    ```ts
    // public/lib/store/greetings/greetings.facade.ts
    import { BaseMultiEntityFacade } from '@redactie/utils';

    import { GreetingsApiService, greetingsApiService } from '../../services/greetings';

    import { GreetingsQuery, greetingsQuery } from './greetings.query';
    import { GreetingsStore, greetingsStore } from './greetings.store';

    export class GreetingsFacade extends BaseMultiEntityFacade<
        GreetingsStore,
        GreetingsApiService,
        GreetingsQuery
    > {}

    export const greetingsFacade = new GreetingsFacade(
        greetingsStore,
        greetingsApiService,
        greetingsQuery
    );
    ```

- `/public/lib/store/greetings/index.ts`

    Als laatste stap exporteren we alles vanuit een index.ts file.
    ```ts
    // public/lib/store/greetings/index.ts
    export * from './greetings.model';
    export * from './greetings.store';
    export * from './greetings.query';
    export * from './greetings.facade';
    ```


We hebben nu een Greetings store maar die doet natuurlijk nog niets (behalve wat boilerplate code).

De volgende stap is dus de facade uitbreiden zodat deze een `getGreeting` methode bevat. Deze methode zal begroeting ophalen en in de store steken. De methode zal er ook voor zorgen dat de begroeting niet steeds opnieuw opgehaald wordt als dat niet nodig is.

```ts
// public/lib/store/greetings/greetings.facade.ts
...
export class GreetingsFacade extends BaseMultiEntityFacade<
    GreetingsStore,
    GreetingsApiService,
    GreetingsQuery
> {
    public async getGreeting(reload = false): Promise<void> {
        // Check if there is already a greeting
        const oldGreeting = this.query.getItem(MAIN_ID);
        const oldGreetingValue = this.query.getItemValue(MAIN_ID);

        // Do nothing when greeting is already present and reload is set to false
        if (!reload && oldGreetingValue) {
            return;
        }

        // Create new item in the store if there is none
        if (!oldGreeting) {
            this.store.addItem(MAIN_ID);
        }

        this.store.setItemIsFetching(MAIN_ID, true);

        return this.service
            .getGreeting()
            .then(greeting => {
                this.store.setItemValue(MAIN_ID, greeting);
            })
            .catch(error => {
                this.store.setItemError(MAIN_ID, error);
            })
            .finally(() => this.store.setItemIsFetching(MAIN_ID, false));
    }
}
...
```

Om niet steeds "main" zelf te typen en de kans te reduceren op typo's maken we een aparte const file `/public/lib/store/greetings/greetings.const.ts` aan:

```ts
// public/lib/store/greetings/greetings.const.ts
export const MAIN_ID = 'main';
```

Merk op dat we door `BaseMultiEntity` te gebruiken we reeds een aantal handige functies ter beschikking krijgen zoals `this.query.getItemValue`, `this.store.setItemIsFetching` en `this.store.setItemValue`.

De `MultiEntityStore` gaat er steeds vanuit dat je van dezelfde entiteit meerder instanties wil ophalen.\ Voorlopig hebben we steeds maar 1 begroeting en is een `MultiEntityStore` niet echt nodig. We raden echter aan om toch vanaf het begin al dit type store te gebruiken aangezien het dan veel gemakkelijker wordt om in de toekomst meerdere instanties op te slaan. Voorlopig slaan we de instantie fixed op onder `main`.

## Stap 4: Greetings hook maken

We hebben nu een API service, een store & een facade. De facade kan reeds gebruikt worden om een begroeting op te halen maar er is nog geen manier om deze beschikbaar te stellen binnen een view.

Om dit te doen maken we een eigen hook aan in de file `/public/lib/hooks/useGreeting/useGreeting.ts`. 

```ts
// public/lib/hooks/useGreeting/useGreeting.ts
import { LoadingState, useObservable } from '@redactie/utils';

import { GreetingModel, greetingsFacade, MAIN_ID } from '../../store/greetings';

const useGreeting = (): [GreetingModel | undefined, LoadingState] => {
    const isFetching = useObservable(
        greetingsFacade.selectItemIsFetching(MAIN_ID),
        LoadingState.Loading
    );
    const greeting = useObservable(greetingsFacade.selectItemValue(MAIN_ID));
    const error = useObservable(greetingsFacade.selectItemError(MAIN_ID));

    const fetchingState = error ? LoadingState.Error : isFetching;

    return [greeting, fetchingState];
};

export default useGreeting;
```

En exporteren we deze via een index file `/public/lib/hooks/useGreeting/index.ts`.
```ts
// public/lib/hooks/useGreeting/index.ts
export { default as useGreeting } from './useGreeting';
```

> [!note] Deze stap is in dit scenario strikt gezien niet noodzakelijk maar voor consistentie raden we toch aan om altijd data van een store toch altijd via een hook binnen te halen.

## Stap 5: Begroeting op de pagina renderen

We hebben nu alle voorbereidende stappen ondernomen en kunnen nu de facade en hook gebruiken binnen onze greetings view.

Als eerste laden we ons nieuw aangemaakte hook in.

```ts
// public/lib/views/Greetings/Greetings.tsx
import { useGreeting } from '../../hooks/useGreeting';
...
const Greetings: FC = () => {
    ...
    const [greeting, greetingLoading] = useGreeting();
    ...
};
...
```
Deze hook zorgt ervoor dat we steeds de main greeting binnen krijgen binnen de component.

De greeting moet echter eerst nog opgehaald worden. Voorlopig doen we dit bij het inladen van de pagina door een useEffect als volgt toe te voegen:

```ts
// public/lib/views/Greetings/Greetings.tsx
import { greetingsFacade } from '../../store/greetings';
...
const Greetings: FC = () => {
    ...

    const [greeting, greetingLoading] = useGreeting();

    useEffect(() => {
        greetingsFacade.getGreeting();
    }, []);
    ...
}
```

Nu rest ons alleen nog de greeting te renderen binnen onze view.

```tsx
// public/lib/views/Greetings/Greetings.tsx
...
const Greetings: FC = () => {
    ...
    return (
        <>
            <ContextHeader title={t(MODULE_TRANSLATIONS.GREETINGS_TITLE)}>
                <ContextHeaderTopSection>{breadcrumbs}</ContextHeaderTopSection>
            </ContextHeader>
            <Container>
                <h2>{greeting?.message}</h2>
            </Container>
        </>
    );
}
```

Nu krijgen we op onze pagina het volgende te zien:

![Documentatie van greetings module](../../../assets/greetings-module-welkom.png ':size=700')

Als laatste kunnen we onze container inhoud nog wrappen in een `DataLoader` zodat er een laadicoon te zien is tot de begroeting is ingeladen.

```tsx
// public/lib/views/Greetings/Greetings.tsx
import { DataLoader, ... } from '@redactie/utils';
...
const Greetings: FC = () => {
    ...
    const [greeting, greetingLoading] = useGreeting();

	useEffect(() => {
		greetingsFacade.getGreeting();
	}, []);

	const renderWelkom = (): ReactElement => <h2>{greeting?.message}</h2>

	return (
		<>
			<ContextHeader title={t(MODULE_TRANSLATIONS.GREETINGS_TITLE)}>
				<ContextHeaderTopSection>{breadcrumbs}</ContextHeaderTopSection>
			</ContextHeader>
			<Container>
				<DataLoader loadingState={greetingLoading} render={renderWelkom} />
			</Container>
		</>
	);
}
...
```

Voila! Onze greetings module is nu klaar. De module doet uiteraard nog niet zo heel veel maar je hebt op basis van deze gids alle basis concepten van de Redactie & WCM module wel beet.

Happy coding!