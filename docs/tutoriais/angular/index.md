# Dicas de Angular

## Tradução (usando angular-i18n)

`npm install angular-i18n --save`

Edite o arquivo `app.module.ts`

```typescript
import { LOCALE_ID } from '@angular/core';

providers: [
	...
	{provide: LOCALE_ID, useValue: 'pt-br'},
	...
]
```

## Lodash

`npm i -S lodash`

`npm install @types/lodash --save-dev --save-exact`

```typescript
import _ from 'lodash'; 

_.algumacoisa();
```